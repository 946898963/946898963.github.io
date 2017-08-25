---
title: AsyncTask的基本用法
date: 2016-05-31 14:36:10
categories:
- 技术
- Android
tags:
- Android
---
 
为了方便我们在子线程中更新UI，Android提供了AsyncTask。使用AsyncTask可以很方便的从子线程切换到主线程。

AsyncTask是一个抽象类，要使用AsyncTask必须创建一个继承自AsyncTaskd的子类，在继承时我们可以为AsyncTask类指定三个泛型参数，这三个参数的用途如下。
>* 1  Params
在执行 AsyncTask 时需要传入的参数，可用于在后台任务中使用。

>* 2  Progress
后台任务执行时，如果需要在界面上显示当前的进度，则使用这里指定的泛型作为进度单位。

>* 3  Result
当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型。


因此，一个最简单的自定义 AsyncTask 就可以写成如下方式：
```
class DownloadTask extends AsyncTask<Void, Integer, Boolean> {
    ……
}
```
这里我们把 AsyncTask 的第一个泛型参数指定为 Void，表示在执行 AsyncTask 的时候不需要传入参数给后台任务。第二个泛型参数指定为 Integer，表示使用整型数据来作为进度显示单位。第三个泛型参数指定为 Boolean，则表示使用布尔型数据来反馈执行结果。当然，目前我们自定义的 DownloadTask 还是一个空任务，并不能进行任何实际的操作，我们还需要去重写 AsyncTask 中的几个方法才能完成对任务的定制。经常需要去重写的方法
有以下四个。

> *     onPreExecute()

> *     doInBackground(Params...)

> *     onProgressUpdate(Progress...)

> *     onPostExecute(Result)

1 onPreExecute()

这个方法会在后台任务开始执行之前调用，用于进行一些界面上的初始化操作，比如显示一个进度条对话框等。

2 doInBackground(Params...)

这个方法中的所有代码都会在子线程中运行，我们应该在这里去处理所有的耗时任务。任务一旦完成就可以通过return语句来将任务的执行结果返回，如果 AsyncTask 的第三个泛型参数指定的是Void，就可以不返回任务执行结果。注意，在这个方法中是不可以进行 UI 操作的，如果需要更新 UI 元素，比如说反馈当前任务的执行进度，可以调用publishProgress(Progress...)方法来完成。

3 onProgressUpdate(Progress...)

当在后台任务中调用了publishProgress(Progress...)方法后，这个方法就会很快被调用，方法中携带的参数就是在后台任务中传递过来的。在这个方法中可以对 UI 进行操作，利用参数中的数值就可以对界面元素进行相应地更新。

4 onPostExecute(Result)

当后台任务执行完毕并通过return语句进行返回时，这个方法就很快会被调用。返回的数据会作为参数传递到此方法中，可以利用返回的数据来进行一些 UI 操作，比如说提醒任务执行的结果，以及关闭掉进度条对话框等。

因此，一个比较完整的自定义 AsyncTask 就可以写成如下方式：
```
 class DownloadTask extends AsyncTask<Void, Integer, Boolean> {
        @Override
        protected void onPreExecute() {
            progressDialog.show(); // 显示进度对话框
        }
        @Override
        protected Boolean doInBackground(Void... params) {
            try {
                while (true) {
                    int downloadPercent = doDownload(); // 这是一个虚构的方法
                    publishProgress(downloadPercent);
                    if (downloadPercent >= 100) {
                        break;
                    }
                }
            } catch (Exception e) {
                return false;
            }
            return true;
        }
        @Override
        protected void onProgressUpdate(Integer... values) {
            // 在这里更新下载进度
            progressDialog.setMessage("Downloaded " + values[0] + "%");
        }
        @Override
        protected void onPostExecute(Boolean result) {
            progressDialog.dismiss(); // 关闭进度对话框
            // 在这里提示下载结果
            if (result) {
                Toast.makeText(context, "Download succeeded",
                        Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(context, " Download failed",
                        Toast.LENGTH_SHORT).show();
            }
        }
```
在这个 DownloadTask中，我们在doInBackground()方法里去执行具体的下载任务。这个方法里的代码都是在子线程中运行的，因而不会影响到主线程的运行。注意这里虚构了一个doDownload()方法，这个方法用于计算当前的下载进度并返回，我们假设这个方法已经存在了。在得到了当前的下载进度后，下面就该考虑如何把它显示到界面上了，由于doInBackground()方法是在子线程中运行的，在这里肯定不能进行 UI 操作，所以我们可以调用 publishProgress()方法并将当前的下载进度传进来，这样onProgressUpdate()方法就会很快被调用，在这里就可以进行 UI 操作了。当下载完成后， doInBackground()方法会返回一个布尔型变量，这样 onPostExecute()方
法就会很快被调用，这个方法也是在主线程中运行的。然后在这里我们会根据下载的结果来弹出相应的 Toast 提示，从而完成整个 DownloadTask 任务。简单来说，使用AsyncTask的诀窍就是，在doInBackground()方法中去执行具体的耗时任务，在 onProgressUpdate()方法中进行 UI 操作，在 onPostExecute()方法中执行一些任务的收尾工作。如果想要启动这个任务，只需编写以下代码即可：
```
new DownloadTask().execute();
```
以上就是 AsyncTask 的基本用法，我们并不需要去考虑什么异步消息处理机制，也不需要专门使用一个 Handler 来发送和接收消息，只需要调用一下publishProgress()方法就可以轻松地从子线程了。

在使用AsyncTask的时候有些需要注意的地方：
> * AsyncTask这个类必须在主线程中加载，也就是说第一次访问AsynaTask必须在主线中，在Android4.1极其以上的版本中已经被这个过程已经被系统自动完成了。在


