# S3TC DXT1/DXT5 Texture Decompression

This is the reposting of some S3TC DXT texture decompression routines that I wrote and published to an old blog in 2009.

## Why republish?

I hadn't thought about these routines in a very long time, but I recently discovered my name in some Amazon Fire copyright notices and traced the notice back to these routines, or a derivative thereof, being include in [ffmpeg](https://github.com/FFmpeg/FFmpeg/blob/master/libavcodec/texturedsp.c), which I thought was pretty nifty. Completely coincidentally, the very next day I received an email asking about these very same routines. I figured that was a sign that I should share these routines again via a more modern mechanism.

## (Dual) License

When using these routines, feel free to chose one of either:

* My original worded "license" (which was intended to be essentially public domain).
* The MIT license 

My original worded "license" was intended to be extremely permissive and does not require attribution. However, given it is highly non-standard (not checked over by a lawyer) then that may lead to ambiguity. If you need something a bit more solid then the MIT license ought to meet your needs.

## Original Blog Post

_October 21st, 2009_

> Earlier this year I offered to write the [S3TC](http://en.wikipedia.org/wiki/S3_Texture_Compression) decompression routines that allowed texture exporting functionality to be included in the ‘Collada Exporter’ plug-in for the [C4 Engine](http://www.terathon.com/c4engine/index.php). After doing a bit of research I was pointed towards the [perfect resource](http://www.opengl.org/registry/specs/EXT/texture_compression_s3tc.txt) by C4’s creator, Eric Lengyel. Any information I’d found prior to this made the process sound a bit convoluted but after looking over the reference I was able to write the decompression routines with relative ease.
> 
> I basically forgot about these routines until a couple of days ago when the question of how to decompress S3TC (specifically DXT1 and DXT5) textures was brought up on the [XNA Creators Club](http://creators.xna.com/) forums. As a result I’ve decided to release the methods that I wrote. In order to ensure the code compiled fine in any project I’ve removed all the C4 specific data-types and methods, and I’ve made no use of any methods that aren’t included. As a result things became a little messier, hopefully the code is still understandable. Without further ado here are my C++ methods to decompress DXT1 and DXT5 textures:
> 
> I’m still in the midst of writing an entry on iPhone, ASP .NET, Linux and Mono so if you’re at all interested check back here in a few days.

**Note:** _Although the C++ routines are reproduced below (as in the original blog post), they're also available in `s3tc.cpp`._

```cpp
// --------------------------------------------------------------------------------
// S3TC DXT1 / DXT5 Texture Decompression Routines
// Author: Benjamin Dobell - http://www.glassechidna.com.au
//
// Feel free to use these methods in any open-source, freeware or commercial
// projects. It's not necessary to credit me however I would be grateful if you
// chose to do so. I'll also be very interested to hear what projects make use of
// this code. Feel free to drop me a line via the contact form on the Glass Echidna
// website.
//
// NOTE: The code was written for a little endian system where sizeof(long) == 4.
// --------------------------------------------------------------------------------
 
// unsigned long PackRGBA(): Helper method that packs RGBA channels into a single 4 byte pixel.
//
// unsigned char r:		red channel.
// unsigned char g:		green channel.
// unsigned char b:		blue channel.
// unsigned char a:		alpha channel.
 
unsigned long PackRGBA(unsigned char r, unsigned char g, unsigned char b, unsigned char a)
{
	return ((r << 24) | (g << 16) | (b << 8) | a);
}
 
// void DecompressBlockDXT1(): Decompresses one block of a DXT1 texture and stores the resulting pixels at the appropriate offset in 'image'.
//
// unsigned long x:						x-coordinate of the first pixel in the block.
// unsigned long y:						y-coordinate of the first pixel in the block.
// unsigned long width: 				width of the texture being decompressed.
// unsigned long height:				height of the texture being decompressed.
// const unsigned char *blockStorage:	pointer to the block to decompress.
// unsigned long *image:				pointer to image where the decompressed pixel data should be stored.
 
void DecompressBlockDXT1(unsigned long x, unsigned long y, unsigned long width, const unsigned char *blockStorage, unsigned long *image)
{
	unsigned short color0 = *reinterpret_cast<const unsigned short *>(blockStorage);
	unsigned short color1 = *reinterpret_cast<const unsigned short *>(blockStorage + 2);
 
	unsigned long temp;
 
	temp = (color0 >> 11) * 255 + 16;
	unsigned char r0 = (unsigned char)((temp/32 + temp)/32);
	temp = ((color0 & 0x07E0) >> 5) * 255 + 32;
	unsigned char g0 = (unsigned char)((temp/64 + temp)/64);
	temp = (color0 & 0x001F) * 255 + 16;
	unsigned char b0 = (unsigned char)((temp/32 + temp)/32);
 
	temp = (color1 >> 11) * 255 + 16;
	unsigned char r1 = (unsigned char)((temp/32 + temp)/32);
	temp = ((color1 & 0x07E0) >> 5) * 255 + 32;
	unsigned char g1 = (unsigned char)((temp/64 + temp)/64);
	temp = (color1 & 0x001F) * 255 + 16;
	unsigned char b1 = (unsigned char)((temp/32 + temp)/32);
 
	unsigned long code = *reinterpret_cast<const unsigned long *>t(blockStorage + 4);
 
	for (int j=0; j < 4; j++)
	{
		for (int i=0; i < 4; i++)
		{
			unsigned long finalColor = 0;
			unsigned char positionCode = (code >>  2*(4*j+i)) & 0x03;
 
			if (color0 > color1)
			{
				switch (positionCode)
				{
					case 0:
						finalColor = PackRGBA(r0, g0, b0, 255);
						break;
					case 1:
						finalColor = PackRGBA(r1, g1, b1, 255);
						break;
					case 2:
						finalColor = PackRGBA((2*r0+r1)/3, (2*g0+g1)/3, (2*b0+b1)/3, 255);
						break;
					case 3:
						finalColor = PackRGBA((r0+2*r1)/3, (g0+2*g1)/3, (b0+2*b1)/3, 255);
						break;
				}
			}
			else
			{
				switch (positionCode)
				{
					case 0:
						finalColor = PackRGBA(r0, g0, b0, 255);
						break;
					case 1:
						finalColor = PackRGBA(r1, g1, b1, 255);
						break;
					case 2:
						finalColor = PackRGBA((r0+r1)/2, (g0+g1)/2, (b0+b1)/2, 255);
						break;
					case 3:
						finalColor = PackRGBA(0, 0, 0, 255);
						break;
				}
			}
 
			if (x + i < width)
				image[(y + j)*width + (x + i)] = finalColor;
		}
	}
}
 
// void BlockDecompressImageDXT1(): Decompresses all the blocks of a DXT1 compressed texture and stores the resulting pixels in 'image'.
//
// unsigned long width:					Texture width.
// unsigned long height:				Texture height.
// const unsigned char *blockStorage:	pointer to compressed DXT1 blocks.
// unsigned long *image:				pointer to the image where the decompressed pixels will be stored.
 
void BlockDecompressImageDXT1(unsigned long width, unsigned long height, const unsigned char *blockStorage, unsigned long *image)
{
	unsigned long blockCountX = (width + 3) / 4;
	unsigned long blockCountY = (height + 3) / 4;
	unsigned long blockWidth = (width < 4) ? width : 4;
	unsigned long blockHeight = (height < 4) ? height : 4;
 
	for (unsigned long j = 0; j < blockCountY; j++)
	{
		for (unsigned long i = 0; i < blockCountX; i++) DecompressBlockDXT1(i*4, j*4, width, blockStorage + i * 8, image);
		blockStorage += blockCountX * 8;
	}
}
 
// void DecompressBlockDXT5(): Decompresses one block of a DXT5 texture and stores the resulting pixels at the appropriate offset in 'image'.
//
// unsigned long x:						x-coordinate of the first pixel in the block.
// unsigned long y:						y-coordinate of the first pixel in the block.
// unsigned long width: 				width of the texture being decompressed.
// unsigned long height:				height of the texture being decompressed.
// const unsigned char *blockStorage:	pointer to the block to decompress.
// unsigned long *image:				pointer to image where the decompressed pixel data should be stored.
 
void DecompressBlockDXT5(unsigned long x, unsigned long y, unsigned long width, const unsigned char *blockStorage, unsigned long *image)
{
	unsigned char alpha0 = *reinterpret_cast<const unsigned char *>(blockStorage);
	unsigned char alpha1 = *reinterpret_cast<const unsigned char *>(blockStorage + 1);
 
	const unsigned char *bits = blockStorage + 2;
	unsigned long alphaCode1 = bits[2] | (bits[3] << 8) | (bits[4] << 16) | (bits[5] << 24);
	unsigned short alphaCode2 = bits[0] | (bits[1] << 8);
 
	unsigned short color0 = *reinterpret_cast<const unsigned short *>(blockStorage + 8);
	unsigned short color1 = *reinterpret_cast<const unsigned short *>(blockStorage + 10);	
 
	unsigned long temp;
 
	temp = (color0 >> 11) * 255 + 16;
	unsigned char r0 = (unsigned char)((temp/32 + temp)/32);
	temp = ((color0 & 0x07E0) >> 5) * 255 + 32;
	unsigned char g0 = (unsigned char)((temp/64 + temp)/64);
	temp = (color0 & 0x001F) * 255 + 16;
	unsigned char b0 = (unsigned char)((temp/32 + temp)/32);
 
	temp = (color1 >> 11) * 255 + 16;
	unsigned char r1 = (unsigned char)((temp/32 + temp)/32);
	temp = ((color1 & 0x07E0) >> 5) * 255 + 32;
	unsigned char g1 = (unsigned char)((temp/64 + temp)/64);
	temp = (color1 & 0x001F) * 255 + 16;
	unsigned char b1 = (unsigned char)((temp/32 + temp)/32);
 
	unsigned long code = *reinterpret_cast<const unsigned long *>(blockStorage + 12);
 
	for (int j=0; j < 4; j++)
	{
		for (int i=0; i < 4; i++)
		{
			int alphaCodeIndex = 3*(4*j+i);
			int alphaCode;
 
			if (alphaCodeIndex <= 12)
			{
				alphaCode = (alphaCode2 >> alphaCodeIndex) & 0x07;
			}
			else if (alphaCodeIndex == 15)
			{
				alphaCode = (alphaCode2 >> 15) | ((alphaCode1 << 1) & 0x06);
			}
			else // alphaCodeIndex >= 18 && alphaCodeIndex <= 45
			{
				alphaCode = (alphaCode1 >> (alphaCodeIndex - 16)) & 0x07;
			}
 
			unsigned char finalAlpha;
			if (alphaCode == 0)
			{
				finalAlpha = alpha0;
			}
			else if (alphaCode == 1)
			{
				finalAlpha = alpha1;
			}
			else
			{
				if (alpha0 > alpha1)
				{
					finalAlpha = ((8-alphaCode)*alpha0 + (alphaCode-1)*alpha1)/7;
				}
				else
				{
					if (alphaCode == 6)
						finalAlpha = 0;
					else if (alphaCode == 7)
						finalAlpha = 255;
					else
						finalAlpha = ((6-alphaCode)*alpha0 + (alphaCode-1)*alpha1)/5;
				}
			}
 
			unsigned char colorCode = (code >> 2*(4*j+i)) & 0x03;
 
			unsigned long finalColor;
			switch (colorCode)
			{
				case 0:
					finalColor = PackRGBA(r0, g0, b0, finalAlpha);
					break;
				case 1:
					finalColor = PackRGBA(r1, g1, b1, finalAlpha);
					break;
				case 2:
					finalColor = PackRGBA((2*r0+r1)/3, (2*g0+g1)/3, (2*b0+b1)/3, finalAlpha);
					break;
				case 3:
					finalColor = PackRGBA((r0+2*r1)/3, (g0+2*g1)/3, (b0+2*b1)/3, finalAlpha);
					break;
			}
 
			if (x + i < width)
				image[(y + j)*width + (x + i)] = finalColor;
		}
	}
}
 
// void BlockDecompressImageDXT5(): Decompresses all the blocks of a DXT5 compressed texture and stores the resulting pixels in 'image'.
//
// unsigned long width:					Texture width.
// unsigned long height:				Texture height.
// const unsigned char *blockStorage:	pointer to compressed DXT5 blocks.
// unsigned long *image:				pointer to the image where the decompressed pixels will be stored.
 
void BlockDecompressImageDXT5(unsigned long width, unsigned long height, const unsigned char *blockStorage, unsigned long *image)
{
	unsigned long blockCountX = (width + 3) / 4;
	unsigned long blockCountY = (height + 3) / 4;
	unsigned long blockWidth = (width < 4) ? width : 4;
	unsigned long blockHeight = (height < 4) ? height : 4;
 
	for (unsigned long j = 0; j < blockCountY; j++)
	{
		for (unsigned long i = 0; i < blockCountX; i++) DecompressBlockDXT5(i*4, j*4, width, blockStorage + i * 16, image);
		blockStorage += blockCountX * 16;
	}
}
```
