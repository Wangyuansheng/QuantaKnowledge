在上篇我总结了关于多线程使用的一些基本知识，看完应该可以对异步消息处理有一定的了解并且可以简单地使用了。不过Android还提供了另外一些好用的工具——AsyncTask。尽管对异步消息处理不怎么熟悉，也可以十分简单地通过使用AsyncTask来从子线程切换到主线程。当然了AsyncTask背后的原理实际上也是异步消息处理机制的，只是Android帮我们做了很好的封装而已。

为了更好的使用AsyncTask，我们先来了解一下它。AsyncTask是一个抽象类，我们要使用它需要新建自己的类来继承他并且重写它的一些方法。在继承的时候我们可以指定3个[泛型][1]参数，这三个参数用途如下：

 - Params：在执行AsyncTask是需要传入的参数，可用于在后台任务中使用。
 - Progress：后台任务执行时，如果需要在界面上显示当前的进度，则使用这里置顶的泛型作为进度单位。
 - Result：当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型。

然后我们就可以这样来定义一个自己的类

```
class MyTask extends AsyncTask<void, Integer, Boolean>{
	...
}
```
这里我定义了第一个是void，意思是不需要传入参数，如果需要也可以更改为其他，第二个为Integer，表示用整型数来作为进度的单位，而Result则指定为Boolean，表示返回结果为布尔值（true/false）

然后我们就需要重写一下它的一些方法了，这里我列举一下4个经常需要重写的四个方法：

 1. onPreExecute() 
这个方法会再后台任务开始之前调用，用于进行一些界面上的初始化操作，比如显示一个进度条对话框表示后台任务正在进行。
 2. doInBackground(Params...)
	这个方法看名字就知道了吧，我们的耗时操作应该放在这里面来处理。不过需要注意的是这里不可以进行UI操作，若需要，比如反馈当前任务的进度，可以借助下面的onProgressUpdate(Progress...)来完成。
 3. onProgressUpdate(Progress...)
	 当在后台任务中调用publishProgress(Progress...)之后，onProgressUpdate(Progress...)方法就会很快地被调用，这个方法中携带的参数就是后台任务中传递过来的，在这里可以对UI进行操作。7
 4. onPostExecute(Result...)
	 当后台任务执行完并return返回时，这个方法就很快被调用了。返回的数据会作为参数传递到此方法中，可以在这里进行UI操作，比如关掉进度条对话框或者提醒任务完成了。

基本了解之后我们就可以这样来定义一个比较完整的AsyncTask了（假设是下载功能的）

```
class DownloadTask extends AsyncTask<Void, Integer, Boolean> {

    @Override
    protected void onPreExecute() {
        progressDialog.show();//显示进度对话框，假设已定义。
    }

    @Override
    protected Boolean doInBackground(Void... params) {
        try{
            while (true){
                int downloadPercent = doDownload();//这是一个虚构的方法,返回下载的进度
                publishProgress(downloadPercent);
                if (downloadPercent >= 100){
                    break;
                }
            }
        }catch (Exception e){
            return false;
        }
        return true;
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        //UI操作，更新下载进度
        progressDialog.setMessage("Download" + values[0] + "%");
    }

    @Override
    protected void onPostExecute(Boolean aBoolean) {
        progressDialog.dismiss();//关闭进度对话框
        if (aBoolean){
            Toast.makeText(context,"Download succeeded",Toast.LENGTH_SHORT).show();
        }else {
            Toast.makeText(context,"Download failed",Toast.LENGTH_SHORT).show();
        }
    }
}
```

要启动这个任务，只需一行：

```
new DownloadTask().execute();
```

以上就是AsyncTask的基本用法了。简直比Handler方便多了有木有。

好了就写这么多吧。

>注：本文内容均是《第二行代码》(第二版)知识。

[1]:http://baike.baidu.com/item/%E6%B3%9B%E5%9E%8B2