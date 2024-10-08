# TestHeifEncoding

һ����λͼ����� HEIF ͼ���ļ���ʾ��. \
A sample of encoding a bitmap to heif image file.

```c#
// ���� HEIF ������
HeifContext context = new HeifContext();
var encoder = context.GetEncoder(HeifCompressionFormat.Hevc);
encoder.SetParameter("preset", "veryfast");

// ��ȡͼ��
var bitmap = SKBitmap.Decode(File.ReadAllBytes("Assets/qwq.jpg"));

// ��������Ҫ�� RGBA ͨ����ͼ�� (libheif ��Ҫ���)
var rgbaBitmap = new SKBitmap(bitmap.Width, bitmap.Height, SKColorType.Rgba8888, SKAlphaType.Premul);
var rgbaBitmapCanvas = new SKCanvas(rgbaBitmap);
rgbaBitmapCanvas.DrawBitmap(bitmap, new SKPoint(0, 0), null);

// ����Ҫ����� HEIF ͼ��
var image = new HeifImage(rgbaBitmap.Width, rgbaBitmap.Height, HeifColorspace.Rgb, HeifChroma.InterleavedRgba32);
image.AddPlane(HeifChannel.Interleaved, bitmap.Width, bitmap.Height, 8);

// �������ݵ�
var sourceData = rgbaBitmap.GetPixels();
var data = image.GetPlane(HeifChannel.Interleaved);
for (int i = 0; i < rgbaBitmap.Height; i++)
{
    NativeMemory.Copy((void*)(sourceData + rgbaBitmap.RowBytes * i), (void*)(data.Scan0 + data.Stride * i), (nuint)Math.Min(rgbaBitmap.RowBytes, data.Stride));
}

// ����ļ�
using var outputFile = File.Create("qwq.heic");
context.EncodeImage(image, encoder);
context.WriteToStream(outputFile);
```