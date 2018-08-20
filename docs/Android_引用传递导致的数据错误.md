Java方法中的参数传递，分为值传递和引用传递（有的书中说Java只有值传递，严格来说是正确的，但并不利于初学者理解）。

其中**值传递**代表的是 ‘基本数据类型的值‘ 的传递，**引用传递**即是 ‘对象实例的地址值‘ 的传递。

／／引用传递经典示例

尽管自以为足够清楚相关概念，这两天工作过程中仍因此导致应用实际运行效果出错，于是决定记录下来，给自己提个醒。
如下所示，利用fragment中的listview展示数据，其中具体的数据通过mOperator来获取，然后交给mAdapter进行展示。

由于应用操作中包含前进后退逻辑，并且可能包含多个数据源，所以通过mShownFilesDataMap保存当前界面需要展示的数据，然后所有数据以数据源的路径作为key存储在mSavedFilesDataMap中。

直接看 updateFilesData() 方法，此方法是mOperator获取到数据后调用的，将其整理为map传递过来。
注意！这里就是开篇所说的引用传递，传递的是这个参数指向的地址值。
那么这个地址对应的对象实例是谁呢，看下一张图。
```
public class ZipBrowserFragment extends BaseFragment {
...
    @Override
    protected void initData() {
        mOperator = new ZipBrowserOperator(this);
        mAdapter = new ZipFileListAdapter(this, mShownFilesDataMap, mFilesDataList, mMotionListener);
        mGridView.setAdapter(mAdapter);
        mListView.setAdapter(mAdapter);
        initBroadcastReceiver();
    }
...
    public void showZipFilesList(String zipPath, String openedFolder, boolean isSetHistory) {
        mZipPath = zipPath;
        if (isSetHistory) {
            mOperator.doShowFilesList(zipPath);
            mActivity.setHistory(new PathBean(zipPath, "/"));
        } else {
            mShownFilesDataMap = mSavedFilesDataMaps.get(zipPath);
            mAdapter.updateFilesData(mShownFilesDataMap, openedFolder);
        }
    }

    public void updateFilesData(Map<String, List<CompressedBean>> filesData) {
        mShownFilesDataMap = filesData;
        mSavedFilesDataMaps.put(mZipPath, filesData);
        mAdapter.updateFilesData(mShownFilesDataMap);
    }
...
}
```

在成员变量中初始化了一个HashMap存储数据，当fragment调用doShowFilesList() 方法时，在子线程中执行耗时操作，当操作结束时调用handleResult() 方法处理数据，然后放入map传递给fragment。
一个数据源的数据获取就完成了。这个过程并没有问题，然而当第二个数据源到来需要mOperator处理时怎么样呢？

由于mFilesDataMap是成员变量，并且完成了初始化，那么它就指向一个HashMap的对象实例所在的地址。只要mFilesDataMap不重新指向其他对象实例的地址 即没有类似mFilesDataMap＝new HashMap<>()的操作，那么这个类中就只存在唯一的一个HashMap对象。

当第一个数据源的数据保存在mFilesDataMap中，传递给fragment的实际是上述唯一对象实例的地址。
当第二个数据源的数据得到后，清空mFilesDataMap然后重新填充数据再交给fragment，传递给fragment的实际上还是那个唯一对象实例的地址。

这边已经清楚了，两次传递传递的是同一个地址值，对应的对象实例经历了一次内容的清空和再次填充。
而对于fragment，两次调用mSavedFilesDataMap.put() 方法，value中保存的也是同一个地址值。指向的对象实例也是同一个，经历了上述清空然后填充的过程。

运行应用后的效果就是，从历史2后退到历史1时发现展示的仍是历史2的数据。
```
public class ZipBrowserOperator {
...
    private Map<String, List<CompressedBean>> mFilesDataMap = new HashMap<>();
...
    public ZipBrowserOperator doShowFilesList(String zipPath) {
...
        startCommand();
    }

    public void startCommand() {
...
        new AsyncTask<Void, Void, Void>() {
            String result;

            @Override
            protected Void doInBackground(Void... voids) {
                result = ZipUtils.executeCommandGetStream(mCommand);
                return null;
            }

            @Override
            protected void onPostExecute(Void aVoid) {
                super.onPostExecute(aVoid);
...
                    handleResult(result);
...
            }
        }.execute();
    }

    public void handleResult(String result) {
...
        mFilesDataMap.clear();
        resolveFileInfoString(result);
        mFragment.updateFilesData(mFilesDataMap);
    }
...
    }
```

这个问题逻辑搞清楚了，也很容易解决。
fragment中需要保存不同的数据源对象，operator中的mFilesDataMap也不能一直指向唯一的对象实例，当每次新的数据源来到时创建一个新的对象实例就可以了。
```
    private Map<String, List<CompressedBean>> mFilesDataMap;
...
    public ZipBrowserOperator doShowFilesList(String zipPath) {
...
        mFilesDataMap = new HashMap<>();
        startCommand();
        return this;
    }
...
```
