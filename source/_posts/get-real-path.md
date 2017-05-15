---
title: Android 根据 Uri 获取文件路径
date: 2016-03-12 10:09:17
categories: Android
tags: Android
---

之前做开发的时候，需要调用系统相册，然后将文件上传，因为业务需要，这个过程就需要将获得的 Uri 转换为 File，用了网上的方法，但是无论如何都没有办法获取到真正的 File Path，试了多种方法以后，利用 Google，输入英文，才在一篇英文博客上找到答案（搜索尤其是技术类的还是要用 Google）。原来之前网上的方法都是 API19 以及 19 以下的，而我的测试机是 API21 的，也难怪不能用了，在此贴出方法，给需要的人一个借鉴。
``` java
import android.annotation.TargetApi;
import android.content.Context;
import android.database.Cursor;
import android.net.Uri;
import android.os.Build;
import android.provider.DocumentsContract;
import android.provider.MediaStore;


public class RealFilePathUtil {

    @TargetApi(Build.VERSION_CODES.KITKAT)
    public static String getRealPathFromURI_API19(Context context, Uri uri){

        String filePath = "";
        String wholeID = DocumentsContract.getDocumentId(uri);
        String id = wholeID.split(":")[1];

        String[] column = {MediaStore.Images.Media.DATA};

        String sel = MediaStore.Images.Media._ID + "=?";

        Cursor cursor = context.getContentResolver().query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                column, sel, new String[]{id}, null);

        int columnIndex = cursor.getColumnIndex(column[0]);

        if (cursor.moveToFirst()) {
            filePath = cursor.getString(columnIndex);
        }
        cursor.close();
        return filePath;
    }


    public static String getRealPathFromURI_API11to18(Context context, Uri contentUri) {
        Cursor cursor = null;
        try {
            String[] proj = {MediaStore.Images.Media.DATA};
            cursor = context.getContentResolver().query(contentUri, proj, null, null, null);

            cursor.moveToFirst();
            int column_index = cursor.getColumnIndex(proj[0]);
            return cursor.getString(column_index);
        } finally {
            if (cursor != null) {
                cursor.close();
            }
        }
    }

}
```

# 更新
---

在经过多机型测试以后，发现了上述方法不足之处，若用户从下载，文件管理（非图册的路径选择图片），返回的Uri就不同，上面的方法就不能用。以下是在 [stackoverflow](https://stackoverflow.com) 上面找到的解决方法，亲测有用，下面直接贴出了代码：

``` java
/**
 * Get a file path from a Uri. This will get the the path for Storage Access
 * Framework Documents, as well as the _data field for the MediaStore and
 * other file-based ContentProviders.
 *
 * @param context The context.
 * @param uri The Uri to query.
 * @author paulburke
 */
public static String getPath(final Context context, final Uri uri) {

    final boolean isKitKat = Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT;

    // DocumentProvider
    if (isKitKat && DocumentsContract.isDocumentUri(context, uri)) {
        // ExternalStorageProvider
        if (isExternalStorageDocument(uri)) {
            final String docId = DocumentsContract.getDocumentId(uri);
            final String[] split = docId.split(":");
            final String type = split[0];

            if ("primary".equalsIgnoreCase(type)) {
                return Environment.getExternalStorageDirectory() + "/" + split[1];
            }

            // TODO handle non-primary volumes
        }
        // DownloadsProvider
        else if (isDownloadsDocument(uri)) {

            final String id = DocumentsContract.getDocumentId(uri);
            final Uri contentUri = ContentUris.withAppendedId(
                    Uri.parse("content://downloads/public_downloads"), Long.valueOf(id));

            return getDataColumn(context, contentUri, null, null);
        }
        // MediaProvider
        else if (isMediaDocument(uri)) {
            final String docId = DocumentsContract.getDocumentId(uri);
            final String[] split = docId.split(":");
            final String type = split[0];

            Uri contentUri = null;
            if ("image".equals(type)) {
                contentUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
            } else if ("video".equals(type)) {
                contentUri = MediaStore.Video.Media.EXTERNAL_CONTENT_URI;
            } else if ("audio".equals(type)) {
                contentUri = MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
            }

            final String selection = "_id=?";
            final String[] selectionArgs = new String[] {
                    split[1]
            };

            return getDataColumn(context, contentUri, selection, selectionArgs);
        }
    }
    // MediaStore (and general)
    else if ("content".equalsIgnoreCase(uri.getScheme())) {
        return getDataColumn(context, uri, null, null);
    }
    // File
    else if ("file".equalsIgnoreCase(uri.getScheme())) {
        return uri.getPath();
    }

    return null;
}

/**
 * Get the value of the data column for this Uri. This is useful for
 * MediaStore Uris, and other file-based ContentProviders.
 *
 * @param context The context.
 * @param uri The Uri to query.
 * @param selection (Optional) Filter used in the query.
 * @param selectionArgs (Optional) Selection arguments used in the query.
 * @return The value of the _data column, which is typically a file path.
 */
public static String getDataColumn(Context context, Uri uri, String selection,
        String[] selectionArgs) {

    Cursor cursor = null;
    final String column = "_data";
    final String[] projection = {
            column
    };

    try {
        cursor = context.getContentResolver().query(uri, projection, selection, selectionArgs,
                null);
        if (cursor != null && cursor.moveToFirst()) {
            final int column_index = cursor.getColumnIndexOrThrow(column);
            return cursor.getString(column_index);
        }
    } finally {
        if (cursor != null)
            cursor.close();
    }
    return null;
}


/**
 * @param uri The Uri to check.
 * @return Whether the Uri authority is ExternalStorageProvider.
 */
public static boolean isExternalStorageDocument(Uri uri) {
    return "com.android.externalstorage.documents".equals(uri.getAuthority());
}

/**
 * @param uri The Uri to check.
 * @return Whether the Uri authority is DownloadsProvider.
 */
public static boolean isDownloadsDocument(Uri uri) {
    return "com.android.providers.downloads.documents".equals(uri.getAuthority());
}

/**
 * @param uri The Uri to check.
 * @return Whether the Uri authority is MediaProvider.
 */
public static boolean isMediaDocument(Uri uri) {
    return "com.android.providers.media.documents".equals(uri.getAuthority());
}
```

参考链接
- [Android | Display Selected Image and Its Real Path](http://hmkcode.com/android-display-selected-image-and-its-real-path/)  
- [Android Gallery on KitKat returns different Uri for Intent.ACTION_GET_CONTENT](http://stackoverflow.com/questions/19834842/android-gallery-on-kitkat-returns-different-uri-for-intent-action-get-content)
