<h1 align = "center"> AXIOM Remote UI Challenge: 2 bit image drawing </h1>

<h2> Subtask Completed in 2 Bit Image Drawing </h2>
<li>A <a href = "https://github.com/eppisai/IMAGE_TO_ARRAY/blob/master/script.py">Python Script</a> to Convert Image to Array</li>
<li><a href = "https://github.com/eppisai/AXIOM-Remote/blob/5506c75e4cac068b5f20bc8ba7200613ca415c2d/Firmware/UI/Painter/Painter.cpp#L628">DrawIcon2bit method</a> in Painter to draw 2 Bit icons in firmware</li>
<li><a href = "https://github.com/eppisai/AXIOM-Remote/blob/21df13431b2eac1c2bfe0043ebd69c182f2fb3bc/Firmware/UI/Widgets/Icon2bit.h#L13">Extended Icon.h</a>, to better fit 2 bit icon data, and its colors</li>
<li>Added <a href = "https://github.com/eppisai/AXIOM-Remote/blob/5506c75e4cac068b5f20bc8ba7200613ca415c2d/Firmware/UI/Painter/Painter.cpp#L658">Transparency feature</a> in Drawing 2 bit Icon</li>
<li><a href = "https://github.com/eppisai/AXIOM-Remote/blob/08281a742191944015bdff802168d39def9ca15a/FirmwareTest/PainterTest.cpp#L217">Unit Test</a> to verify Proper Implementation</li>


<h2> How Image to array conversion in Python Script is happening? </h2>
 <li><a href = "https://github.com/eppisai/IMAGE_TO_ARRAY"> Repository for Image to Array Conversion Python Script</a></li>
 
 ```
def image_to_array(im):
    im = rgba_to_2bit(im)
    array = []
    im = np.array(im)
    im = im.reshape(im.size//4,4)
    setbits = np.array([0, 2, 4, 6])
    for i in im:
        bits = i<<setbits
        array.append(bits.sum())
    return array
  ```
  
  Above image_to_array Function,
  1. converts the image, to its quantized version with 4 colors, 
  2. then converts image to a numpy array, which is then reshaped to a 2d numpy array of width 4 and height (im.size/4) 
  3. We reshaped the array into size of 4 since, each byte in final output would have 4 pixel, (2 bit per pixel)
  4. For each pixel row (size is 4) in image array, we convert it to byte, shifting each bit in muliples of 2(4 values), and then adding them
  5. We set the 1st bit of each row of image as Least significant bit, because we need to start drawing from 1st bit. (last bit is MSB - most significant bit)
  6. made a new array with bytes of pixel bits. 
  
  <br>
  
  ```
  def rgba_to_2bit(image):
    white_image = Image.new('RGBA', image.size, (255, 255, 255))
    image = Image.alpha_composite(white_image, image)
    image = image.convert('L')
    image = image.quantize(4)
    return image
  
  ```
  
  
  Above rgba_to_2bit Function,
  1. Mixes background color (white to get correct rgb conversion) with pixel color in the ratio of their alpha value(transparency), called alpha compositing.
  2. converts the image to greyscale and then in quantizes its colors to 4.
  3. We are converting image to grey scale to have a gradual decreasing shades of black to white, we can extract colors manually also.

<br>

 <li><a href = "https://github.com/eppisai/AXIOM-Remote/tree/2bitimageTask"> Repository of AXIOM Remote, with 2 bit image Task</a></li>
<h2> DrawIcon2bit Method </h2>

```
void Painter::DrawIcon2Bit(const Icon2bit* image, uint16_t x, uint16_t y, float transparency = 0)
{
        int b = 0;
        for (uint16_t yIndex = 0; yIndex < image->Height; yIndex++)
        {
            uint16_t yPos = y + yIndex;

            for (uint16_t xIndex = 0; xIndex < image->Width;)
            {
                uint8_t data = image->Data[b];
                for (int i = 0; i < 8; i+=2)
                {
                    uint8_t pixel = ((1 << 2) - 1) & (data >> i);

                    if(pixel)
                    {
                        uint16_t pixelcolor = image->IndexedColors[pixel];
                        uint16_t backcolor = _framebuffer[y * _framebufferWidth + x];
                        uint16_t finalcolor = ApplyTransparency(transparency, pixelcolor, backcolor);
                        DrawPixel(x + xIndex, yPos, finalcolor);
                    }

                    xIndex++;
                }
                b++;
            }
        }  
}

```

Above is a farily straight forward function, which processes each byte and extracts bits from bytes in pairs of two and matches each pair to its corresponding color, and then apply Transparency to it.

<br>

<h2> Transparency function for 2 bit icon </h2>

```
uint16_t Painter::ApplyTransparency(float transparency, uint16_t color, uint16_t background)
{   
    uint16_t pixelColor = 0;

    //Red color channel with transparency amount of Background color
    pixelColor +=  uint16_t( (color & 0xF800)*(1 - transparency) + (background & 0xF800)*transparency ) & 0xF800;

    //Green color channel with transparency amount of Background color
    pixelColor += uint16_t( (color & 0x07E0)*(1 - transparency) + (background & 0x07E0)*transparency ) & 0x07E0;

    //blue color channel with transparency amount of Background color
    pixelColor += uint16_t( (color & 0x001F)*(1 - transparency) + (background & 0x001F)*transparency ) & 0x001F;

    return pixelColor;
} 

```

Above function,applies transparency by taking background color in Transparency amount, and Pixel color in (1 - transparency) amount in the new color. <br>Note : Value of Transparency is between 0 and 1 (float, not double).I am using RGB565 color chanel masks, complete article is in references.

<br>

<h2> Extended Icon2bit.h </h2>

```
#ifndef ICON2BIT_H
#define ICON2BIT_H

#include <inttypes.h>

#include "../../Utils.h"
enum class ImageDataFormat : uint8_t
{
	Monochrome = 0,
	Indexed2Bit = 1
};

struct Icon2bit
{
    const uint16_t Width;
    const uint16_t Height;
    ImageDataFormat Format;
    uint16_t IndexedColors[4];
    const uint8_t* Data;
};

#endif // ICON2BIT_H

```

Above structure, extends Icon.h with color storage of 2 bit icon and, and enum for user readability

<h2 align = "center"> References Used </h2>
<ui>
	<li>For color extraction for 16 bit RGB565 pixel, I am using RGB565 color masks, as descrided <a href = "https://docs.microsoft.com/en-us/windows/win32/directshow/working-with-16-bit-rgb">here</a> </li>
	<li>For RGB to RGBA conversion and for transparency in remote (alpha compositing), I am using <a href = "https://stackoverflow.com/questions/2049230/convert-rgba-color-to-rgb">this</a> simple formula, described in 1st answer</li>
</ui>

<br>

## Output
* 1st image is 2 bit icon with 0 percent transparency and 2nd image is 2 bit icon with 60 percent transparency

  <img align = "center" src="https://user-images.githubusercontent.com/54789531/114575495-d9065e00-9c97-11eb-9e43-0212168c6fd5.png" width="350" title="with zero transparency"><br>

  <img align = "center" src="https://user-images.githubusercontent.com/54789531/114575650-018e5800-9c98-11eb-85de-e493e168ea96.jpg" width="350" title="with 60 percent transparency"><br>




