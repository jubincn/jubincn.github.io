## Suffix is not enough
We used to judge a file type by its suffix. For example, .gif for gif format, .jpg for jpeg and .png for png files. However, this is not a reliable way to detect a file type since we can change file suffix to any name we want. Check file header is a more preferable to detect file type.

## File header
There is a file header for every common used image file, we can detect file type using it. The popular file header is listed below:

* Extended WebP
```Extended WebP
/**
 * Each VP8X WebP image has "features" byte following its ChunkHeader('VP8X')
 */
private static final int EXTENDED_WEBP_HEADER_LENGTH = 21;
```

* Simple WebP
```Java
/**
 * Each WebP header should consist of at least 20 bytes and start
 * with "RIFF" bytes followed by some 4 bytes and "WEBP" bytes.
 * More detailed description if WebP can be found here:
 * <a href="https://developers.google.com/speed/webp/docs/riff_container">
 *   https://developers.google.com/speed/webp/docs/riff_container</a>
 */
private static final int SIMPLE_WEBP_HEADER_LENGTH = 20;
```

* JPEG
```Java
private static final byte[] JPEG_HEADER = new byte[] {(byte) 0xFF, (byte)0xD8, (byte)0xFF}
```

* PNG
```Java
private static final byte[] PNG_HEADER = new byte[] {
      (byte) 0x89,
      'P', 'N', 'G',
      (byte) 0x0D, (byte) 0x0A, (byte) 0x1A, (byte) 0x0A};
```

* GIF
```Java
private static final byte[] GIF_HEADER_87A = ImageFormatCheckerUtils.asciiBytes("GIF87a");
private static final byte[] GIF_HEADER_89A = ImageFormatCheckerUtils.asciiBytes("GIF89a");

public static byte[] asciiBytes(String value) {
  Preconditions.checkNotNull(value);
  try {
    return value.getBytes("ASCII");
  } catch (UnsupportedEncodingException uee) {
    // won't happen
    throw new RuntimeException("ASCII not found!", uee);
  }
}
```

* BMP_HEADER_LENGTH
```BMP
/**
 * Every bmp image starts with "BM" bytes
 */
private static final byte[] BMP_HEADER = ImageFormatCheckerUtils.asciiBytes("BM");
```

## Detect file type with Fresco
There is a helper class in Fresco which can help detect image file type with an input stream. We can use that util class in our project:
```
/**
    * @return {@link com.facebook.imageformat.DefaultImageFormats}
    */
   @NonNull
   public static ImageFormat getImageFormat(@NonNull ContentResolver cr, @NonNull Uri imageUri) {
       if (cr == null || imageUri == null) {
           return ImageFormat.UNKNOWN;
       }

       ImageFormat result = null;
       try {
           InputStream imageInputStream = cr.openInputStream(imageUri);
           if (imageInputStream != null) {
               result = ImageFormatChecker.getImageFormat(imageInputStream);
           }
       } catch (FileNotFoundException e) {
           Log.printErrStackTrace(TAG, e,
                   "FileNotFoundException in getImageFormat, uri: " + imageUri);
       } catch (IOException e) {
           Log.printErrStackTrace(TAG, e, "IOException in getImageFormat, uri: " + imageUri);
       }

       return (result == null) ? ImageFormat.UNKNOWN : result;
   }
```

## Detect file type with plain codes
If Fresco is not available, we can write our own util class like those in Fresco. Let's go through source codes of Fresco and find out how things works.
1. getImageFormat is a static method, which calls instance method determineImageFormat(InputStream)
```Java
/**
 * Tries to read up to MAX_HEADER_LENGTH bytes from InputStream is and use read bytes to
 * determine type of the image contained in is. If provided input stream does not support mark,
 * then this method consumes data from is and it is not safe to read further bytes from is after
 * this method returns. Otherwise, if mark is supported, it will be used to preserve original
 * content of is.
 * @param is
 * @return ImageFormat matching content of is InputStream or UNKNOWN if no type is suitable
 * @throws IOException if exception happens during read
 */
public static ImageFormat getImageFormat(final InputStream is) throws IOException {
  return getInstance().determineImageFormat(is);
}
```
2. determineImageFormat do the hard jobs. It read header from input stream and determine the ImageFormat.
```Java
public ImageFormat determineImageFormat(final InputStream is) throws IOException {
  Preconditions.checkNotNull(is);
  final byte[] imageHeaderBytes = new byte[mMaxHeaderLength];
  final int headerSize = readHeaderFromStream(mMaxHeaderLength, is, imageHeaderBytes);

  if (mCustomImageFormatCheckers != null) {
    for (ImageFormat.FormatChecker formatChecker : mCustomImageFormatCheckers) {
      ImageFormat format = formatChecker.determineFormat(imageHeaderBytes, headerSize);
      if (format != null && format != ImageFormat.UNKNOWN) {
        return format;
      }
    }
  }
  ImageFormat format = mDefaultFormatChecker.determineFormat(imageHeaderBytes, headerSize);
  if (format == null) {
    format = ImageFormat.UNKNOWN;
  }
  return format;
}
```
3. There is a default implementation to calculate mMaxHeaderLength, which covers most commonly used image format.
```Java
/**
  * Maximum header size for any image type.
  *
  * <p>This determines how much data {@link ImageFormatChecker#getImageFormat(InputStream) reads
  * from a stream. After changing any of the type detection algorithms, or adding a new one, this
  * value should be edited.
  */
 final int MAX_HEADER_LENGTH = Ints.max(
     EXTENDED_WEBP_HEADER_LENGTH,
     SIMPLE_WEBP_HEADER_LENGTH,
     JPEG_HEADER_LENGTH,
     PNG_HEADER_LENGTH,
     GIF_HEADER_LENGTH,
     BMP_HEADER_LENGTH);

@Override
public int getHeaderSize() {
   return MAX_HEADER_LENGTH;
}
```

4. Read header into a byte array with maximum header size. We can be sure that the header info is stored in that byte array.
```Java
/**
 * Reads up to maxHeaderLength bytes from is InputStream. If mark is supported by is, it is
 * used to restore content of the stream after appropriate amount of data is read.
 * Read bytes are stored in imageHeaderBytes, which should be capable of storing
 * maxHeaderLength bytes.
 * @param maxHeaderLength the maximum header length
 * @param is
 * @param imageHeaderBytes
 * @return number of bytes read from is
 * @throws IOException
 */
private static int readHeaderFromStream(
    int maxHeaderLength,
    final InputStream is,
    final byte[] imageHeaderBytes)
    throws IOException {
  Preconditions.checkNotNull(is);
  Preconditions.checkNotNull(imageHeaderBytes);
  Preconditions.checkArgument(imageHeaderBytes.length >= maxHeaderLength);

  // If mark is supported by the stream, use it to let the owner of the stream re-read the same
  // data. Otherwise, just consume some data.
  if (is.markSupported()) {
    try {
      is.mark(maxHeaderLength);
      return ByteStreams.read(is, imageHeaderBytes, 0, maxHeaderLength);
    } finally {
      is.reset();
    }
  } else {
    return ByteStreams.read(is, imageHeaderBytes, 0, maxHeaderLength);
  }
}
```

5. We can set our custom ImageFormatChecker to detect those image format not in the list. And if our custom checker failed, the default ImageFormatChecker will still work.
```java
if (mCustomImageFormatCheckers != null) {
  for (ImageFormat.FormatChecker formatChecker : mCustomImageFormatCheckers) {
    ImageFormat format = formatChecker.determineFormat(imageHeaderBytes, headerSize);
    if (format != null && format != ImageFormat.UNKNOWN) {
      return format;
    }
  }
}
```

6. Detect image format with header byte and header size.
```Java
/**
 * Tries to match imageHeaderByte and headerSize against every known image format. If any match
 * succeeds, corresponding ImageFormat is returned.
 *
 * @param headerBytes the header bytes to check
 * @param headerSize the available header size
 * @return ImageFormat for given imageHeaderBytes or UNKNOWN if no such type could be recognized
 */
@Nullable
@Override
public final ImageFormat determineFormat(byte[] headerBytes, int headerSize) {
  Preconditions.checkNotNull(headerBytes);

  if (WebpSupportStatus.isWebpHeader(headerBytes, 0, headerSize)) {
    return getWebpFormat(headerBytes, headerSize);
  }

  if (isJpegHeader(headerBytes, headerSize)) {
    return DefaultImageFormats.JPEG;
  }

  if (isPngHeader(headerBytes, headerSize)) {
    return DefaultImageFormats.PNG;
  }

  if (isGifHeader(headerBytes, headerSize)) {
    return DefaultImageFormats.GIF;
  }

  if (isBmpHeader(headerBytes, headerSize)) {
    return DefaultImageFormats.BMP;
  }

  return ImageFormat.UNKNOWN;
}
```

* Check is WebP
```Java
/**
 * Checks if imageHeaderBytes contains WEBP_RIFF_BYTES and WEBP_NAME_BYTES and if the
 * header is long enough to be WebP's header.
 * WebP file format can be found here:
 * <a href="https://developers.google.com/speed/webp/docs/riff_container">
 *   https://developers.google.com/speed/webp/docs/riff_container</a>
 * @param imageHeaderBytes image header bytes
 * @return true if imageHeaderBytes contains a valid webp header
 */
public static boolean isWebpHeader(
    final byte[] imageHeaderBytes,
    final int offset,
    final int headerSize) {
  return headerSize >= SIMPLE_WEBP_HEADER_LENGTH &&
      matchBytePattern(imageHeaderBytes, offset, WEBP_RIFF_BYTES) &&
      matchBytePattern(imageHeaderBytes, offset + 8, WEBP_NAME_BYTES);
}
```

* Check is Jpeg
```Java
/**
 * Checks if imageHeaderBytes starts with SOI (start of image) marker, followed by 0xFF.
 * If headerSize is lower than 3 false is returned.
 * Description of jpeg format can be found here:
 * <a href="http://www.w3.org/Graphics/JPEG/itu-t81.pdf">
 *   http://www.w3.org/Graphics/JPEG/itu-t81.pdf</a>
 * Annex B deals with compressed data format
 * @param imageHeaderBytes
 * @param headerSize
 * @return true if imageHeaderBytes starts with SOI_BYTES and headerSize >= 3
 */
private static boolean isJpegHeader(final byte[] imageHeaderBytes, final int headerSize) {
  return headerSize >= JPEG_HEADER.length &&
      ImageFormatCheckerUtils.startsWithPattern(imageHeaderBytes, JPEG_HEADER);
}
```

* Check is PNG
```Java
/**
 * Checks if array consisting of first headerSize bytes of imageHeaderBytes
 * starts with png signature. More information on PNG can be found there:
 * <a href="http://en.wikipedia.org/wiki/Portable_Network_Graphics">
 *   http://en.wikipedia.org/wiki/Portable_Network_Graphics</a>
 * @param imageHeaderBytes
 * @param headerSize
 * @return true if imageHeaderBytes starts with PNG_HEADER
 */
private static boolean isPngHeader(final byte[] imageHeaderBytes, final int headerSize) {
  return headerSize >= PNG_HEADER.length &&
      ImageFormatCheckerUtils.startsWithPattern(imageHeaderBytes, PNG_HEADER);
}
```

* Check is GIF
```Java
/**
 * Checks if first headerSize bytes of imageHeaderBytes constitute a valid header for a gif image.
 * Details on GIF header can be found <a href="http://www.w3.org/Graphics/GIF/spec-gif89a.txt">
 *  on page 7</a>
 * @param imageHeaderBytes
 * @param headerSize
 * @return true if imageHeaderBytes is a valid header for a gif image
 */
private static boolean isGifHeader(final byte[] imageHeaderBytes, final int headerSize) {
  if (headerSize < GIF_HEADER_LENGTH) {
    return false;
  }
  return ImageFormatCheckerUtils.startsWithPattern(imageHeaderBytes, GIF_HEADER_87A) ||
      ImageFormatCheckerUtils.startsWithPattern(imageHeaderBytes, GIF_HEADER_89A);
}
```

* Check is BMP
```Java
/**
 * Checks if first headerSize bytes of imageHeaderBytes constitute a valid header for a bmp image.
 * Details on BMP header can be found <a href="http://www.onicos.com/staff/iz/formats/bmp.html">
 * </a>
 * @param imageHeaderBytes
 * @param headerSize
 * @return true if imageHeaderBytes is a valid header for a bmp image
 */
private static boolean isBmpHeader(final byte[] imageHeaderBytes, final int headerSize) {
  if (headerSize < BMP_HEADER.length) {
    return false;
  }
  return ImageFormatCheckerUtils.startsWithPattern(imageHeaderBytes, BMP_HEADER);
}
```

7. Util Class
```Java
private static boolean matchBytePattern(
    final byte[] byteArray,
    final int offset,
    final byte[] pattern) {
  if (pattern == null || byteArray == null) {
    return false;
  }
  if (pattern.length + offset > byteArray.length) {
    return false;
  }

  for (int i = 0; i < pattern.length; ++i) {
    if (byteArray[i + offset] != pattern[i]) {
      return false;
    }
  }

  return true;
}
```

```Java
/**
 * Checks if byteArray interpreted as sequence of bytes starts with pattern
 * starting at position equal to offset.
 * @param byteArray the byte array to be checked
 * @param pattern the pattern to check
 * @return true if byteArray starts with pattern
 */
public static boolean startsWithPattern(
    final byte[] byteArray,
    final byte[] pattern) {
  Preconditions.checkNotNull(byteArray);
  Preconditions.checkNotNull(pattern);
  if (pattern.length > byteArray.length) {
    return false;
  }

  for (int i = 0; i < pattern.length; ++i) {
    if (byteArray[i] != pattern[i]) {
      return false;
    }
  }

  return true;
}
```

```Java
/**
 * Helper method that transforms provided string into it's byte representation
 * using ASCII encoding.
 * @param value the string to use
 * @return byte array representing ascii encoded value
 */
public static byte[] asciiBytes(String value) {
  Preconditions.checkNotNull(value);
  try {
    return value.getBytes("ASCII");
  } catch (UnsupportedEncodingException uee) {
    // won't happen
    throw new RuntimeException("ASCII not found!", uee);
  }
}
```
