 

# The Display Communication Protocol

## **The serial messages**

Host sends x, y, character, 8bit RGB332 colour.

The x and y are character spaces which are multiplied by 8 on the display. There is a *posleft* and a *postop* variable which determine the origin of the start of the image on the display. If no character or overlay character is needed the character value needs to be sent as 32 (0x20) for a space.

 

## **Special commands if the x co-ordinate byte is set to over 249**

#### Host sends 250, colour LSB, colour MSB, dummy 0.

 This will set the text colour to a different colour than specified by the display code

#### Host sends 251, posleft LSB, posleft MSB, dummy 0.

 This will set the left origin of the start of the image area.

#### Host sends 252, postop LSB, postop MSB, dummy 0.

 This will set the top origin of the start of the image area.

 

## **The feedback byte from a display**

The display code will also send one byte messages to the host.

Display buffer control - The display will indicate a close to full buffer by sending a 0xFF.

The display will send a 0xFE when the buffer has recovered a little.

If the buffer becomes close to full, the host should stop sending image data until the buffer has recovered.

 

## **Screen code operation**

The code operates by processing the serial data when at least 4 bytes have been received in the buffer. Serial transmission of image data is always slow so the way we hope to achieve a good frame rate is by only sending changed pixels. The host needs to do most of the work and needs to take the camera image data and store it in an array big enough to hold the entire image. In addition there needs to be another array as a cache of the main image array. During a scan of the image array and compare with the cache, if there is a difference then the x, y and the converted to RGB332 colour can be sent directly to the display using 0x20 as the value of the character and the cache updated with the new image cell data. Below is an example of how it would be done on an Arduino based on image size 24 x 24

 

## **Host code example**

```
byte SCREEN[576];
byte CACHE[576];
int x, y, ipos;
ipos = 0;

for(y = 0; y < 24; y++)
{
 for(x = 0; x < 24; x++)
 {
  if(SCREEN[ipos] != CACHE[ipos])
  {
   Serial.write(x);
   Serial.write(y);
   Serial.write(0x20);
   Serial.write(SCREEN[ipos]);
   CACHE[ipos] = SCREEN[ipos];
  }
  ipos ++;
 }
}
```

