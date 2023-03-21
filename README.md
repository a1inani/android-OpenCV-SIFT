# android-OpenCV-SIFT
This example implements a computer vision project with Android Native C++ and OpenCV 4.5.1. The feature points of the image will be calculated with the open source SIFT algorithm in 2020.

![](./screenshot/img14.png)

## YouTube Tutorial
[<img src="./screenshot/cover.jpg" width="550px">](https://youtu.be/rIu-mHX_MXM)

## Step 1: 建立專案
Open Android Studio and create a project based on Android Native C++. This option will build a C/C++ environment that can execute programs calling C/C++ language through JNI (Java Native Interface).

![](./screenshot/img01.png)
![](./screenshot/img02.png)


After the creation is complete, we can change the project window to the `Project` folder mode to facilitate subsequent operations.

![](./screenshot/img03.png)

## Step 2: 下載 OpenCV SDK
Enter the OpenCV [official website](https://opencv.org/releases/) to select the version version (this example uses 4.5.1) and download the Android version. After downloading, enter the SDK folder, find the java folder and rename it to `openCVLibrary451` (this action is to import this folder, name it first). To make a digression, when the previous version is imported into the java folder, the system will automatically rename it to the library of the corresponding version number. Personally, I think it is a bug, so it is troublesome to manually process the name (it should be resolved in the future).

![](./screenshot/img04.png)

## Step 3: 匯入 OpenCV SDK

Click the upper toolbar File > New > Import Module. Select the `openCVLibrary451` folder that has been renamed in the second step, click Next and click Finish to import.

![](./screenshot/img05.png)

After the import is complete, you can see that `openCVLibrary451` is in the main directory folder, but what we import is to be used as a module rather than an APP (red box in the figure below). So we have to manually change it to library. (The previous version did not have this problem. There are a lot of problems after version 4.3, but in order to show the latest version of SIFT API, it has to be used)

![](./screenshot/img06.png)


Modify the build.gradle of `openCVLibrary451`. Change OpenCV from application to library to import. Modify `apply plugin: 'com.android.application'` and change `application` to `library`. And delete the applicationId. Delete the following:

```
defaultConfig {
    applicationId "org.opencv
}
```


The modified build.gradle (openCVLibrary451) is as follows:

![](./screenshot/img07.png)

## Step 4: dependencies加入OpenCV

Click File > Project Structure to enter the Project Structure screen, click the Dependencies option on the left, select app for Modules, and click [+] to select `3 Module Dependency`, as shown below:

![](./screenshot/img08.png)

Click on the `openCVLibrary451` module that was just imported earlier. After clicking ok, our project can successfully use the OpenCV library.

![](./screenshot/img09.png)

We can open build.gradle(app) to see if `openCVLibrary451` has been imported successfully.

![](./screenshot/img10.png)

## Step 5: 新增Native Libraries
Copy the `armeabi-v7a` and `x86` files from `\OpenCV-android-sdk\sdk\native\libs` downloaded earlier to `\app\libs`, these two CPU types are usually required. The former is arm7, the current mainstream architecture of mobile phones, and the latter is for developers to debug and execute on the emulator. Until now, there are 7 different CPUs for Android which are ARMv5, ARMv7 (from 2010) x86 (from 2011) MIPS (from 2012) ARMv8, MIPS64 and x86_64 (from 2014). In order to support these CPUs, we need to package the corresponding so files into the apk.

![](./screenshot/img11.png)


Next, specify the `jniLibs` path, open the build.gradle of the app, and add:

```
sourceSets{
    main {
        jniLibs.srcDirs = ['libs']
    }
}
```

![](./screenshot/img12.png)

## activity_main.xml

Add an `ImageView` to display the anchor point results after SIFT feature selection.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/sample_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toTopOf="@+id/sample_text"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.498"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.641"
        app:srcCompat="@drawable/ic_launcher_background" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## Complete MainActivity.java
The variable `inputImage` can freely replace any image in the drawable folder. The next step will teach you how to open the folder and put your own photos.

```java
import androidx.appcompat.app.AppCompatActivity;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Bundle;
import android.widget.ImageView;
import android.widget.TextView;

import org.opencv.android.Utils;
import org.opencv.core.Mat;
import org.opencv.core.MatOfKeyPoint;
import org.opencv.features2d.Features2d;
import org.opencv.features2d.SIFT;
import org.opencv.imgproc.Imgproc;

public class MainActivity extends AppCompatActivity {

    // Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("native-lib");
        System.loadLibrary("opencv_java4");
    }

    private ImageView imageView;
    // make bitmap from image resource
    private Bitmap inputImage; 
    private SIFT sift = SIFT.create();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Example of a call to a native method
        TextView tv = findViewById(R.id.sample_text);
        tv.setText(stringFromJNI());

        inputImage = BitmapFactory.decodeResource(getResources(), R.drawable.coca_cola);
        imageView = (ImageView) this.findViewById(R.id.imageView);
        sift();
    }
    public void sift() {
        Mat rgba = new Mat();
        Utils.bitmapToMat(inputImage, rgba);
        MatOfKeyPoint keyPoints = new MatOfKeyPoint();
        Imgproc.cvtColor(rgba, rgba, Imgproc.COLOR_RGBA2GRAY);
        sift.detect(rgba, keyPoints);
        Features2d.drawKeypoints(rgba, keyPoints, rgba);
        Utils.matToBitmap(rgba, inputImage);
        imageView.setImageBitmap(inputImage);
    }

    /**
     * A native method that is implemented by the 'native-lib' native library,
     * which is packaged with this application.
     */
    public native String stringFromJNI();
}
```

## Add test image
Open app > src > main > res > drawable and put the test picture into this folder.

![](./screenshot/img13.png)

## Error exclusion

If the following error message occurs after compiling and executing:

> java.lang.UnsatisfiedLinkError: dlopen failed: library "libc++_shared.so" not found

Solution: Add in cmake of build.gradle(app):

```
cmake {
    arguments "-DANDROID_STL=c++_shared"
}
```

## Reference
- [Android studio載入OpenCV 4.3 module](https://hjwang520.pixnet.net/blog/post/404969512-android-studio%E8%BC%89%E5%85%A5opencv-4.3-module)
- [How to configure OpenCV in Android Studio 4.1 or higher](https://www.youtube.com/watch?v=-0Yx1UzozzQ)
- [OpenCV Android Installation + SIFT Tutorial 2016 EASY](https://www.youtube.com/watch?v=cLK9CjQ-pNI)
