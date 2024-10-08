# TestHeifEncoding

一个将位图编码成 HEIF 图像文件的示例. \
A sample of encoding a bitmap to heif image file.

```c#
// 创建 HEIF 上下文
HeifContext context = new HeifContext();
var encoder = context.GetEncoder(HeifCompressionFormat.Hevc);
encoder.SetParameter("preset", "veryfast");

// 读取图像
var bitmap = SKBitmap.Decode(File.ReadAllBytes("Assets/qwq.jpg"));

// 制作所需要的 RGBA 通道的图像 (libheif 需要这个)
var rgbaBitmap = new SKBitmap(bitmap.Width, bitmap.Height, SKColorType.Rgba8888, SKAlphaType.Premul);
var rgbaBitmapCanvas = new SKCanvas(rgbaBitmap);
rgbaBitmapCanvas.DrawBitmap(bitmap, new SKPoint(0, 0), null);

// 创建要编码的 HEIF 图像
var image = new HeifImage(rgbaBitmap.Width, rgbaBitmap.Height, HeifColorspace.Rgb, HeifChroma.InterleavedRgba32);
image.AddPlane(HeifChannel.Interleaved, bitmap.Width, bitmap.Height, 8);

// 拷贝数据到
var sourceData = rgbaBitmap.GetPixels();
var data = image.GetPlane(HeifChannel.Interleaved);
for (int i = 0; i < rgbaBitmap.Height; i++)
{
    NativeMemory.Copy((void*)(sourceData + rgbaBitmap.RowBytes * i), (void*)(data.Scan0 + data.Stride * i), (nuint)Math.Min(rgbaBitmap.RowBytes, data.Stride));
}

// 输出文件
using var outputFile = File.Create("qwq.heic");
context.EncodeImage(image, encoder);
context.WriteToStream(outputFile);
```