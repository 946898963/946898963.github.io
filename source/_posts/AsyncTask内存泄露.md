---
title: AsyncTask内存泄露
date: 2016-06-05 17:48:40

categories:
- 技术
- Android
- 内存泄露
tags:
- Android
- 内存泄露
---

AsyncTask在使用过程中可能会出现内存溢出的情况，下面就这种情况就行介绍

AsyncTask的用法很简单，那么我们看下面这段代码

```
private TextView mTv;

new AsyncTask<...> {

    @Override

    protected void onPostExecute(Objecto) {

        mTv.setText("text");

    }

}.execute();
```
乍一看好像没什么问题，但这段代码会导致内存泄露，线程有可能会超出当前Activity的生命周期之后仍然在run，因为这个时候线程已经不受控制了。Activity生命周期已经结束，需要被系统回收掉，但是AsyncTask还在持有TextView的引用，这样就导致了内存泄露。

修改代码
  
```
private TextView mTv;

new AsyncTask<...> {

    @Override

    protected void onPostExecute(Objecto) {

        //mTv.setText("text");

}

}.execute();
```
直接注释掉，不做UI操作了，虽然表面看不会存在问题了，但是实际上还存在着问题，因为这里AsyncTask是个内部类，由于Java内部类的特点，AsyncTask内部类会持有外部类的隐式引用。即使从代码上看我在AsyncTask里没有持有外部的任何引用，但是写在Activity里，对context仍然会有个强引用，这样如果线程超过Activity生命周期，Activity还是无法回收造成内存泄露。
那问题怎么解决呢，有两种办法：
> * 第一，在Activity生命周期结束前，去cancel AsyncTask，因为Activity都要销毁了，这个时候再跑线程，绘UI显然已经没什么意义了。
> * 第二，如果一定要写成内部类的形式，写成静态内部类，同时对context采用WeakRefrence,在使用之前判断是否为空。


示例代码如下：
```
public abstract static class WeakAsyncTask<Params, Progress, Result, WeakTarget> extends AsyncTask<Params, Progress, Result> {
		protected WeakReference<WeakTarget> mTarget;

        public WeakAsyncTask(WeakTarget target) {
            mTarget = new WeakReference<WeakTarget>(target);
        }

        @Override
        protected final void onPreExecute() {
            final WeakTarget target = mTarget.get();
            if (target != null) {
                this.onPreExecute(target);
            }
        }

        @Override
        protected final Result doInBackground(Params... params) {
            final WeakTarget target = mTarget.get();
            if (target != null) {
                return this.doInBackground(target, params);
            } else {
                return null;
            }
        }

        @Override
        protected final void onPostExecute(Result result) {
            final WeakTarget target = mTarget.get();
            if (target != null) {
                this.onPostExecute(target, result);
            }
        }

        @Override
        protected void onProgressUpdate(Progress... values) {
            
            final WeakTarget target = mTarget.get();
            if (target != null) {
                this.onProgressUpdate(target,values);
            }
            
        }
        
        protected abstract void onPreExecute(WeakTarget target);
        
        protected abstract Result doInBackground(WeakTarget target,
                                                 Params... params);

        protected abstract void onPostExecute(WeakTarget target, Result result);

        protected abstract void onProgressUpdate(WeakTarget target,Progress... values);
    }
    
    new WeakAsyncTask<String,String,String,Context>(this) {


            @Override
            protected void onPreExecute(Context context) {

            }

            @Override
            protected String doInBackground(Context context, String... params) {
                return null;
            }



            @Override
            protected void onPostExecute(Context context, String s) {

            }

            @Override
            protected void onProgressUpdate(Context context, String... values) {

            }
        }.execute();    
    
```

参考链接：[内存泄露之Thread][1]


  [1]: http://946898963.github.io/2016/06/05/%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2%E4%B9%8BThread/