## 目录

### MainActivity.java
```
    ......
    private void initView() {
        mRecyclerView = (RecyclerView) findViewById(R.id.recycle_view);
        mAdapter = new LibrariesAdapter(this, items);
        mRecyclerView.setAdapter(mAdapter);
        mLLManager = new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);
        mRecyclerView.setLayoutManager(mLLManager); 
        mRecyclerView.addOnItemTouchListener(new OnItemTouchListener());
    }
 
    /* 切换布局样式 */
    public void switchDisplayStyle() {
        isListDisplayStyle = !isListDisplayStyle;
        if (isListDisplayStyle()) {
            mRecyclerView.setLayoutManager(mLLManager);
            return;
        }
        refreshGridLayoutManager();
        mRecyclerView.setLayoutManager(mGLManager);
    }

    /* 当窗口尺寸变化时更新grid列数 */
    private void refreshGridLayoutManager() {
        if (mGLManager == null) {
            mGLManager = new GridLayoutManager(this, 1);
            setGridSpans();
            return;
        }
        if (hasWindowSizeChanged) {
            setGridSpans();
            hasWindowSizeChanged = false;
        }
    }

    private void setGridSpans() {
        // 获取可用宽度，得到适配列数
        if (mAvailableWidth == 0 || hasWindowSizeChanged) {
            mAvailableWidth = getWindow().getDecorView().getDisplay().getWidth();
        }
        final int spanCount = mAvailableWidth / getResources()
                .getDimensionPixelSize(R.dimen.grid_item_width);
        mGLManager.setSpanCount(spanCount);
        // 根据不同条件设置item占用不同列数
        mGLManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
            @Override
            public int getSpanSize(int position) {
                return mAdapter.isGroupItem(position) ? spanCount : 1;
            }
        });
    }

    /* 监听窗口尺寸变化 */
    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        if (mGLManager != null) {
            hasWindowSizeChanged = true;
            refreshGridLayoutManager();
        }
    }

    /* 触摸事件处理 */
    private class OnItemTouchListener extends RecyclerView.SimpleOnItemTouchListener {
        private final int DOUBLE_CLICK_INTERNAL = 500;
        private long touchTime;

        @Override
        public void onTouchEvent(RecyclerView rv, MotionEvent e) {
            switch (e.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    if (e.getButtonState() == MotionEvent.BUTTON_SECONDARY) {  //右键菜单
                        new MenuDialog(SeafileMainActivity.this).show(e.getX(), e.getY());
                        return;
                    }
                    if (e.getButtonState() == MotionEvent.BUTTON_PRIMARY) { //左键选中，进入
                        View v = rv.findChildViewUnder(e.getX(), e.getY());
                        if (v != null) {
                            int pos = rv.getChildAdapterPosition(v);
                            if (isCtrlPressed) {
                                mAdapter.switchItemSelected(pos);  //ctrl多选
                            } else {
                                mAdapter.setOnlySelectedItem(pos);  //非ctrl排他单选
                            }
                            if (mLastPosition == pos && System.currentTimeMillis() - touchTime
                                    < DOUBLE_CLICK_INTERNAL && !mAdapter.isGroupItem(pos)) {  //双击判定
                                enter(pos);
                            }
                            touchTime = System.currentTimeMillis();
                            mLastPosition = pos;
                            return;
                        }
                    }
                    break;
            }
        }

        /* 必须重写，监听才能生效，暂不明白 */
        @Override
        public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {
            onTouchEvent(rv, e);
            return false;
        }
    }

    private class MenuDialog extends Dialog implements View.OnClickListener {
...

        /* 重写方法，根据坐标显示弹出位置 */
        public void show(float x, float y) {
            Window window = this.getWindow();
            if (window != null) {
                WindowManager.LayoutParams lp = window.getAttributes();
                window.setGravity(Gravity.START | Gravity.TOP);
                lp.dimAmount = 0.0f;  //背景阴影，1.0为最暗
                lp.x = (int) x;  //设置window布局坐标
                lp.y = (int) y;
                window.setAttributes(lp);
                show();
            }
        }
    }
......

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        switch (keyCode) {
            case KeyEvent.KEYCODE_CTRL_LEFT:
            case KeyEvent.KEYCODE_CTRL_RIGHT:
                isCtrlPressed = true;  //ctrl按下
                break;
            case KeyEvent.KEYCODE_DPAD_UP:
            case KeyEvent.KEYCODE_DPAD_LEFT:
                if (mLastPosition != 0) {
                    mAdapter.setOnlySelectedItem(--mLastPosition);
                }
                break;
            case KeyEvent.KEYCODE_DPAD_DOWN:
            case KeyEvent.KEYCODE_DPAD_RIGHT:
                if (mLastPosition != mAdapter.getItemCount()) {
                    mAdapter.setOnlySelectedItem(++mLastPosition);
                }
                break;
            case KeyEvent.KEYCODE_ENTER:
            case KeyEvent.KEYCODE_NUMPAD_ENTER:
                enter(mLastPosition);
                break;
            case KeyEvent.KEYCODE_ESCAPE:
                break;
            case KeyEvent.KEYCODE_A:
                if (isCtrlPressed) {
                    mAdapter.selectAllItems();
                }
                break;
        }
        return super.onKeyDown(keyCode, event);
    }

    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {

        switch (keyCode) {
            case KeyEvent.KEYCODE_CTRL_LEFT:
            case KeyEvent.KEYCODE_CTRL_RIGHT:
                isCtrlPressed = false;  //ctrl释放
                break;
        }
        return super.onKeyUp(keyCode, event);
    }

    public boolean isListDisplayStyle() {
        return isListDisplayStyle;
    }

```

### Adapter
```
  public class LibrariesAdapter extends RecyclerView.Adapter<LibrariesAdapter.ViewHolder> {

    private SeafileMainActivity mActivity;
    private List<SeafItem> mItems;
    // SparseArray 内部二分查找，适用于千以下的数据量，SparseBooleanArray类似，只是value类型有限制；
    private SparseBooleanArray mSelectedPositions = new SparseBooleanArray();
    private List<SeafItem> mSelectedItems = new ArrayList<>();
    // 定义多种数据类型，可以保证ViewHolder复用时只复用同样类型的，避免视图错乱；
    public static final int TYPE_LIST_GROUP = 0;
    public static final int TYPE_GRID_GROUP = 1;
    public static final int TYPE_LIST_REPO = 2;
    public static final int TYPE_LIST_DIRENT = 3;
    public static final int TYPE_GRID_REPO = 4;
    public static final int TYPE_GRID_DIRENT = 5;

    public LibrariesAdapter(SeafileMainActivity activity, List<SeafItem> items) {
        mActivity = activity;
        mItems = items;
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view;
        // 根据不同数据类型加载多种布局，此处为三种布局样式；
        if (viewType == TYPE_LIST_GROUP || viewType == TYPE_GRID_GROUP) {
            view = LayoutInflater
                    .from(mActivity).inflate(R.layout.list_item_group, parent, false);
        } else if (mActivity.isListDisplayStyle()) {
            view = LayoutInflater
                    .from(mActivity).inflate(R.layout.list_item_list, parent, false);
        } else {
            view = LayoutInflater
                    .from(mActivity).inflate(R.layout.list_item_grid, parent, false);
        }
        return new ViewHolder(view, viewType);
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        SeafItem item = mItems.get(position);
        holder.tvTitle.setText(item.getTitle());
        //根据条件进行不同布局的holder赋值；
        ......
    }

    @Override
    public int getItemViewType(int position) {
        if (mItems.get(position) instanceof SeafGroup) {
            return mActivity.isListDisplayStyle() ? TYPE_LIST_GROUP : TYPE_GRID_GROUP;
        } else if (mItems.get(position) instanceof SeafRepo) {
            return mActivity.isListDisplayStyle() ? TYPE_LIST_REPO : TYPE_GRID_REPO;
        } else {
            return mActivity.isListDisplayStyle() ? TYPE_LIST_DIRENT : TYPE_GRID_DIRENT;
        }
    }

    static class ViewHolder extends RecyclerView.ViewHolder {
        ImageView imgIcon, imgStatus;
        TextView tvTitle, tvSize, tvTime;
        View itemView;

        ViewHolder(View itemView, int viewType) {
            super(itemView);
            //不同数据类型-->不同布局类型-->不同初始化
            if (viewType == TYPE_LIST_GROUP || viewType == TYPE_GRID_GROUP) {
                tvTitle = itemView.findViewById(R.id.tv_file_title);
            } else {...}
        }

    }

    public void setItemSelected(int position, boolean selected) {
        if (selected) {
            mSelectedPositions.put(position, true);
            mSelectedItems.add(mItems.get(position));
        } else {
            mSelectedPositions.delete(position);
            mSelectedItems.remove(position);
        }
        notifyItemChanged(position);
    }

    public void switchItemSelected(int position) {
        if (mSelectedPositions.get(position)) {
            mSelectedPositions.delete(position);
            mSelectedItems.remove(mItems.get(position));
        } else {
            mSelectedPositions.put(position, true);
            mSelectedItems.add(mItems.get(position));
        }
        notifyItemChanged(position);
    }

    public void setOnlySelectedItem(int pos) {
        mSelectedPositions.clear();
        mSelectedItems.clear();
        mSelectedItems.add(mItems.get(pos));
        mSelectedPositions.put(pos, true);
        notifyDataSetChanged();
    }

    public void selectAllItems() {
        for (int i = 0; i < mItems.size(); i++) {
            mSelectedPositions.put(i, true);
            mSelectedItems.addAll(mItems);
        }
        notifyDataSetChanged();
    }

    public void deselectAllItems() {
        mSelectedPositions.clear();
        mSelectedItems.clear();
        notifyDataSetChanged();
    }
}
```
