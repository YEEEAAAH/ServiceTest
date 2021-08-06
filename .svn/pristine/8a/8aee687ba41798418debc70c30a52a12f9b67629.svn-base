package cn.itcast.servicetest;

import android.app.Notification;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.app.PendingIntent;
import android.app.Service;
import android.content.Intent;
import android.graphics.BitmapFactory;
import android.graphics.Color;
import android.os.Binder;
import android.os.Build;
import android.os.Build.VERSION;
import android.os.Environment;
import android.os.FileObserver;
import android.os.IBinder;
import android.util.Log;
import android.widget.Toast;

import java.io.File;

import androidx.core.app.NotificationBuilderWithBuilderAccessor;
import androidx.core.app.NotificationCompat;

public class DownloadService extends Service {
    private  DownloadBinder mBinder = new DownloadBinder();
    private static final String TAG = "DownloadService";
    private DownloadTask downloadTask;
    private String downloadUrl;






    private DownloadListener listener = new DownloadListener() {
        @Override
        public void onProgress(int progress) {
            getNotificationManager().notify(1,getNotification("Downloading...",progress));
        }

        @Override
        public void onSuccess() {
            Log.d(TAG,"onSuccess");
            downloadTask = null;
            //下载成功前台服务关闭通知，创建一个下载成功的通知
            stopForeground(true);
            getNotificationManager().notify(1,getNotification("Download Success",-1));
            Toast.makeText(DownloadService.this,"下载成功",Toast.LENGTH_LONG).show();
        }

        @Override
        public void onFailed() {
            Log.d(TAG,"onFailed");
            downloadTask = null;
            stopForeground(true);
            getNotificationManager().notify(1,getNotification("Download Failed",-1));
            Toast.makeText(DownloadService.this,"下载失败",Toast.LENGTH_LONG).show();
        }

        @Override
        public void onPaused() {
            Log.d(TAG,"onPaused");
            downloadTask =null;
            //Toast.makeText(DownloadService.this,"暂停",Toast.LENGTH_LONG).show();
        }

        @Override
        public void onCanceled() {
            Log.d(TAG,"onCanceled");
        downloadTask = null;
        stopForeground(true);
        Toast.makeText(DownloadService.this,"取消",Toast.LENGTH_LONG).show();
        }
    };

    class DownloadBinder extends Binder{
        public void startDownload(String url){
            if(downloadTask==null){
                downloadUrl =url;
                downloadTask =new DownloadTask(listener);
                downloadTask.execute(downloadUrl);
                startForeground(1,getNotification("Downloading...",0));
                Toast.makeText(DownloadService.this,"Downloading...",Toast.LENGTH_LONG).show();
            }
        }
        public void pauseDownload(){
            if(downloadTask!=null){
                downloadTask.pauseDownload();
            }
        }
        public void cancelDownload(){
            if(downloadTask!=null){
                downloadTask.cancelDownload();
            }else{
                if(downloadUrl!=null){
                    //取消下载时删除文件，关闭通知
                    String fileName = downloadUrl.substring(downloadUrl.lastIndexOf("/"));
                    String directory = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).getPath();
                    File file = new File(directory+fileName);
                    if(file.exists()){

                        file.delete();
                    }
                    getNotificationManager().cancel(1);
                    stopForeground(true);
                    //Toast.makeText(DownloadService.this,"取消",Toast.LENGTH_LONG).show();
                }
            }
        }
    }

    public DownloadService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        return (IBinder) mBinder;
    }
    private NotificationManager getNotificationManager(){

        return (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
    }
    private Notification getNotification(String title,int progress){
        Intent intent =new Intent(this,MainActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this,0,intent,0);
        String CHANNEL_ONE_ID = "cn.itcast.servicetest";
        String CHANNEL_ONE_NAME = "Channel One";
        NotificationChannel notificationChannel = null;
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
            notificationChannel = new NotificationChannel(CHANNEL_ONE_ID,CHANNEL_ONE_NAME,NotificationManager.IMPORTANCE_HIGH);
            notificationChannel.enableLights(true);
            notificationChannel.setLightColor(Color.RED);
            notificationChannel.setShowBadge(true);
            notificationChannel.setLockscreenVisibility(Notification.VISIBILITY_PUBLIC);
            NotificationManager manager = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
            manager.createNotificationChannel(notificationChannel);

        }
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this).setChannelId(CHANNEL_ONE_ID);
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher));
        builder.setContentIntent(pi);
        builder.setContentTitle(title);
        if(progress>0){
            //显示下载进度
            builder.setContentText(progress+"%");
            builder.setProgress(100,progress,false);
        }
        return builder.build();
    }
}