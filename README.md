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
    im = rgba_to_4bit(im)
    array = []
    im = np.array(im)
    im = im.reshape(im.size//4,4)
    setbits = np.array([0, 2, 4, 6])
    for i in im:
        bits = i<<setbits
        array.append(bits.sum())
    return array
  ```
  
  ```
  def rgba_to_4bit(image):
    white_image = Image.new('RGBA', image.size, (255, 255, 255))
    image = Image.alpha_composite(white_image, image)
    image = image.convert('L')
    image = image.quantize(4)
    return image
  
  ```
