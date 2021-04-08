<h1 align = "center"> AXIOM Remote UI Challenge: 2 bit image drawing </h1>

<h2> Subtask Completed in 2 Bit Image Drawing </h2>
<li>A Python Script to Convert Image to Array</li>
<li>DrawIcon2bit method in Painter to draw 2 Bit icons in firmware</li>
<li>Extended Icon.h, to better fit 2 bit icon data, and its colors</li>
<li>Added Transparency feature in Drawing 2 bit Icon</li>
<li>Unit Test to verify Proper Implementation</li>

<h2> How Image to array conversion in Python Script is happening? </h2>
 
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
  
  Above Function,
  1. converts the image, to its quantized version with 4 colors, 
  2. then converts image a numpy array, which is then reshaped to a 2d numpy array of width 4 and height (im.size/4) 
  3. We reshaped the array into size of 4 since, we byte in final output would need to 4 pixel, (2 bit per pixel)
  4. For each pixel row (size is 4) in image array, we convert it to byte, shifting each bit in 2 muliple, and then adding them (i.e OR operation at bit level)
  5. not we set the 1st bit of each row of image as Least significant bit, because we need to start drawing from 1st bit. (last bit MSB - most significant bit)
  6. made a new array with bytes of pixel bits. 
  
  ```
  def rgba_to_2bit(image):
    white_image = Image.new('RGBA', image.size, (255, 255, 255))
    image = Image.alpha_composite(white_image, image)
    image = image.convert('L')
    image = image.quantize(4)
    return image
  
  ```
