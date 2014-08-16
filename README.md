Installation
============

Add this repository to _build.gradle_ in your project's root.

```
allprojects {
    repositories {
        maven {
            url 'https://github.com/nirvanfallacy/RapidDecoder/raw/master/repository'
        }
    }
}
```

Add dependencies to _build.gradle_ in your module.

```
dependencies {
    compile 'rapid.decoder:library:0.1.0'
    compile 'rapid.decoder:jpeg-decoder:0.1.0'
    compile 'rapid.decoder:png-decoder:0.1.0'
}
```

**jpeg-decoder** and **png-decoder** are optional. Refer to [asdf](#basic-decoding).

Basic decoding
==============

To decode a bitmap from resource:

```java
import rapid.decoder.BitmapDecoder;

Bitmap bitmap = BitmapDecoder.from(getResources(), R.drawable.image).decode();
```

Bitmap can also be decoded from other sources like this:

```java
// Decodes bitmap from byte array
byte[] bytes;
Bitmap bitmap = BitmapDecoder.from(bytes).decode();

// Decodes bitmap from file
Bitmap bitmap = BitmapDecoder.from("/sdcard/image.png").decode();

// Decodes bitmap from network
Bitmap bitmap = BitmapDecoder.from("http://server.com/image.jpeg").decode();

// Decodes bitmap from content provider
Bitmap bitmap = BitmapDecoder.from("content://app/user/0/profile").decode();

// Decodes bitmap from other app's resource
Bitmap bitmap = BitmapDecoder.from("android.resource://com.app/drawable/ic_launcher")
                             .decode();

// Decodes bitmap from stream
InputStream is;
Bitmap bitmap = BitmapDecoder.from(is).decode();

// Decodes from database
final SQLiteDatabase db;
Bitmap bitmap = BitmapDecoder
        .from(new Queriable() {
            @Override
            public Cursor query() {
                return db.query("table", new String[]{"column"}, "id=1", null, null,
                        null, null);
            }
        })
        .decode();
```

Advanced decoding
=================

Scaling
-------

All of the scaling operations automatically decode bounds of bitmaps and calculate [inSampleSize](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inSampleSize), so you don't need to consider about them at all.

```java
// Scaling to 400x300
Bitmap bitmap = BitmapDecoder.from(getResouces(), R.drawable.image)
                             .scale(400, 300)
                             .decode();

// Scaling by 50%
Bitmap bitmap = BitmapDecoder.from("/sdcard/image.png")
                             .scaleBy(0.5)
                             .decode();

```

Regional decoding
-----------------

Only partial area of bitmap can be decoded. (Supports down to Froyo)


```java
// Decodes the area (100, 200)-(300, 400) of the bitmap scaling it by 50%.
Bitmap bitmap = BitmapDecoder.from("/sdcard/image.jpeg")
                             .region(100, 200, 300, 400)
                             .scaleBy(0.5)
                             .decode();

```

Mutable decoding
----------------

You can directly modify decoded bitmap if it was decoded as mutable. This also supports down to Froyo.

```java
Bitmap bitmap2 = something;
Bitmap bitmap = BitmapDecoder.from(getResources(), R.drawable.image)
                             .mutable()
                             .decode();
Canvas canvas = new Canvas(bitmap);
canvas.draw(bitmap2, 0, 0, null):
```

Direct drawing
--------------

It is possible to draw bitmap directly to canvas from BitmapDecoder. It's generally faster than drawing bitmap after full decoding.

```java
Bitmap bitmap = Bitmap.createBitmap(width, height, Config.RGB_565);
Canvas canvas = new Canvas(bitmap);
BitmapDecoder.from("/image.png").scaleBy(0.2, 0.5).draw(canvas, x, y);
```

Post processing
---------------

You can hook decoded bitmap and replace it to something you want.

```java
// Make rounded image
Bitmap bitmap = BitmapDecoder.from("http://somewhere.com/image.jpeg")
        .postProcessor(new BitmapPostProcessor() {
                @Override
                public Bitmap process(Bitmap bitmap) {
                    Bitmap bitmap2 = Bitmap.createBitmap(bitmap.getWidth(), bitmap.getHeight(),
                        Bitmap.Config.ARGB_8888);
                    Canvas canvas = new Canvas(bitmap2);
                    
                    Paint paint = new Paint();
                    paint.setColor(0xffffffff);
                    RectF area = new RectF(0, 0, bitmap2.getWidth(), bitmap2.getHeight());
                    canvas.drawRoundRect(area, 10, 10, paint);
                    
                    paint.reset();
                    paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
                    canvas.drawBitmap(bitmap, 0, 0, paint);
                    
                    return bitmap2;
                }
        })
        .decode();
```

In this case you **MUST NOT** recycle the given bitmap.

Framing
=======

You can apply [ScaleType](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html) to make a decoded bitmap fit into certain size.

```java
Bitmap bitmap = BitmapDecoder.from("content://authority/path")
        .frame(frameWidth, frameHeight, ImageView.ScaleType.CENTER_CROP)
        .decode();
```

When the image needs to be cropped, it uses region() internally so that only required area can be decoded. In this reason, it takes less memory and less time than implementing it only in APIs provided by Android.

You can also provide your own custom framing method by extending FramedDecoder and FramingMethod. But note that it's not documened yet.

Caching
=======

