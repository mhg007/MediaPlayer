# MediaPlayer
Activity 3: Create a Media Player using Foreground Service
Create a mediaplayer using foreground service, which will constantly show the song being currently
played in Notification bar.

I have created an activity named MainActivity and Service for creating Notification and ForegroundService.
Code of Main Activity
package com.example.musicplayer

import android.content.Intent
import android.os.Build
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.view.View

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
    fun play(view: View){
        var serviceIntent = Intent(this, MyService::class.java)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) startForegroundService(serviceIntent)
        else startService(serviceIntent)

    }

    fun pause(view: View){
        var serviceIntent = Intent(this, MyService::class.java)
        stopService(serviceIntent)
    }


}

Code of Service named as MyService
package com.example.musicplayer

import android.annotation.SuppressLint
import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.app.Service
import android.content.Intent
import android.media.MediaPlayer
import android.os.Build
import android.os.IBinder
import android.widget.RemoteViews
import android.widget.RemoteViews.RemoteView
import androidx.core.app.NotificationCompat

class MyService : Service() {
    val channelId = "MUSIC_PLAYER"
    val channelName = "Music Player"
    lateinit var largeView : RemoteViews
    lateinit var mediaPlayer : MediaPlayer
    override fun onCreate() {
        super.onCreate()
        mediaPlayer = MediaPlayer.create(this,R.raw.rema)
    }
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        startForeground(1234,createNotification(intent))
        mediaPlayer.start()
        return START_STICKY
    }

    override fun onDestroy() {
        mediaPlayer.pause()
        stopSelf()
        stopForeground(true)
        super.onDestroy()

    }
    override fun onBind(intent: Intent): IBinder {
        TODO("Return the communication channel to the service.")
    }
    private fun createNotificationChannel(){
        val channel = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O)
            NotificationChannel(channelId, channelName, NotificationManager.IMPORTANCE_DEFAULT)
        else TODO("VERSION.SDK_INT < O")
        val manager = getSystemService(NOTIFICATION_SERVICE) as NotificationManager
        manager.createNotificationChannel(channel)
    }
    @SuppressLint("RemoteViewLayout")
    private fun createNotification(intent:Intent?):Notification{
        createNotificationChannel()
        val input = intent!!.getStringExtra("inputExtra")
        val intent = Intent(this,MainActivity::class.java)
        val pendingIntent = intent.let{
            PendingIntent.getActivity(this,0,intent,PendingIntent.FLAG_IMMUTABLE)
        }
        largeView = RemoteViews(packageName,R.layout.music_large)
        val smallView = RemoteViews(packageName, R.layout.music_small)
        return NotificationCompat
            .Builder(this, channelId)
            .setSmallIcon(android.R.drawable.ic_media_play)
            .setStyle(NotificationCompat.DecoratedCustomViewStyle())
            .setCustomContentView(smallView)
            .setCustomBigContentView(largeView)
            .setContentTitle(channelName)
            .setContentText(input)
            .setAutoCancel(true)
            .setContentIntent(pendingIntent)
            .build()

    }
}

Code for xml file of main activity
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    tools:context=".MainActivity">
    <Button
        android:id="@+id/btnPlay"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="play"
        android:text="Play"
        />
    <Button
        android:id="@+id/btnPause"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="pause"
        android:text="Pause"
        />
</LinearLayout>

Code for xml file for large music
<?xml version="1.0" encoding="utf-8"?>
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Music song" />

    <Button
        android:id="@+id/btn"
        android:text="Play"
        android:layout_width="57dp"
        android:layout_height="48dp"
        />
</LinearLayout>

Code for xml file of small music
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Music is currently playing" />
</LinearLayout>

The End
