<div align="center">

## Gcipher


</div>

### Description

Gcipher is a graphical encryption program, that converts a text file into an encrypted bitmap file. When the file is opened in windows it opens as a picture. To see the contents one needs to decrypt the file usnig the same program and password supplied during encryption.
 
### More Info
 
The program takes input from the GUI designed using c++ graphics.

The output filename entered for encryption should have .bmp as extension otherwise the file will not open as a picture in windows.

The program creates a file


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Somil Bhandari](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/somil-bhandari.md)
**Level**          |Intermediate
**User Rating**    |5.0 (15 globes from 3 users)
**Compatibility**  |C\+\+ \(general\)
**Category**       |[Graphics/ Sound](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/graphics-sound__3-15.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/somil-bhandari-gcipher__3-11135/archive/master.zip)





### Source Code

```
/***************************************************************************/
/***************************************************************************/
/***************************************************************************/
/*  A program that takes a file as input from the user and encrypts it.  */
/*     The encryption algorithm used is simple XOR algorithm.     */
/* After encryption the file is saved as a bitmap file which is identified */
/* by the windows as a picture file . The algorithm can be easily reversed */
/* to get the original text again. Since the file is shown as a picture in */
/* windows it is not very easy to identify that it contains encrypted text */
/***************************************************************************/
/*    Sourcecode developed by Somil Bhandari and Swapnil Patil     */
/*   Students of Shri Vaishnav Institute of Technology and Science   */
/*        Third Year Computer Science and Engineering        */
/***************************************************************************/
/***************************************************************************/
/***************************************************************************/
#include<dir.h>
#include<dos.h>
#include<stdio.h>
#include<conio.h>
#include<string.h>
#include<stdlib.h>
#include<fstream.h>
#include<graphics.h>
#include<iostream.h>
#define PASSWORD_SIZE_MAX	18
typedef struct { /* The bitmap file information header */
	unsigned short type;   // Must always be set to 'BM'
				 // to declare that this is a bitmap file
				 // Standard value: 19778
	unsigned long size;    // Specifies the size of file in bytes
	unsigned short reserved1; // Must always be set to zero
	unsigned short reserved2; // Must always be set to zero
	unsigned long offsetBits; // Specifies the offset from the
				 // beginning of the file to the bitmap data
				 // Standard value: 118 for 4-bit bitmap file
} BitmapFileHeader;
typedef struct { /* The image information header */
	unsigned long size; // Specifies the size of BitmapInfoHeader
	// structure in bytes, Standard value: 40
	unsigned long width; // Width of image in pixels
	unsigned long height; // Height of image in pixels
	unsigned short planes; // specifies the no. of planes, Standard value: 1
	unsigned short bitCount; // no. of bits per pixels, in our case it is 4
	unsigned long compression; // type of compression, 0 if no compression
	unsigned long sizeImage; // size of image data in bytes
	long xPelsPerMeter; // horizontal pixels per meter, usually set to zero
	long yPelsPerMeter; // vertical pixels per meter, usually set to zero
	unsigned long colorsUsed; // no of colors used, if zero it is
	// calculated using the bitCount number
	unsigned long colorsImportant; // no of colors important,
	// if zero, all colors are important
} BitmapInfoHeader;
typedef struct { /* The pixel structure */
  unsigned char col[3];
} SinglePixel;
/****************************************************************************/
/*                                     */
/*     		 Class to initialize mouse             */
/*          Mouse Driver needed if used in DOS          */
/*                                     */
/****************************************************************************/
int bt, mx, my; // Global variables for mouse position and button
class Mouse {
	private:
	union REGS i_, o_;
	public:
	void initMouse() { // Initialize Mouse
		i_.x.ax = 0;
		int86(0x33, &i_, &o_);
	}
	void showMouse(void) { // Show Cursor
		gotoxy(30, 20);
		i_.x.ax = 1;
		int86(0x33, &i_, &o_);
	}
	void getMousePosition(int *bt, int *x, int *y) { // Get the cursor position
		i_.x.ax = 3;
		int86(0x33, &i_, &o_);
		*bt = o_.x.bx;
		*x = o_.x.cx;
		*y = o_.x.dx;
	}
	void restrict(int left, int top, int right, int bottom ) {
		i_.x.ax = 7; // Confine cursor in a particular area
		i_.x.cx = left;
		i_.x.dx = right;
		int86(0x33, &i_, &o_);
		i_.x.ax = 8;
		i_.x.cx = top;
		i_.x.dx = bottom;
		int86(0x33, &i_, &o_);
	}
	void hide(void)	{ // Hide the cursor
		i_.x.ax = 2;
		int86(0x33, &i_, &o_);
	}
}m;
/****************************************************************************/
/*                                     */
/*        A class to encrypt and decrypt the files          */
/*                                     */
/****************************************************************************/
class Encryption {
	private:
	int lengthOfPassword_;
	unsigned char blockRead_[PASSWORD_SIZE_MAX], previousBlock_[PASSWORD_SIZE_MAX], afterXor_[PASSWORD_SIZE_MAX];
	public:
	void xorEncrypt(char* toEncrypt, char* outputFile, char* password);
	void xorDecrypt(char* toDecrypt, char* outputFile, char* password);
};
void Encryption::xorEncrypt(char* toEncrypt, char* outputFile, char* password) {
	short count = 0;
	int i;
	long size = 0;
	long side; /* side of the square image (to be calculated) */
	char ch;
	FILE* inFile;
	inFile = fopen(toEncrypt,"rb");
	ofstream outFile(outputFile, ios::binary);
	BitmapFileHeader fileHeader;
	BitmapInfoHeader bitmapHeader;
	SinglePixel pix;
	strcpy(previousBlock_, password);
	lengthOfPassword_ = strlen(password);
	while(!feof(inFile)) {    // Calculate the size of file
		fgetc(inFile);
		size++;
	}
	for(side = 0; (side*side) <= size; side++); // Calculate side of image to be formed
	side--;
	fileHeader.type = 19778;
	fileHeader.size = (side * side) + 118;
	fileHeader.reserved1 = 0;
	fileHeader.reserved2 = 0;
	fileHeader.offsetBits = 118;
	bitmapHeader.size = 40;
	bitmapHeader.width = side;
	bitmapHeader.height = side;
	bitmapHeader.planes = 1;
	bitmapHeader.bitCount = 4;
	bitmapHeader.compression = 0;
	bitmapHeader.sizeImage = (side * side) / 2;
	bitmapHeader.xPelsPerMeter = 0;
	bitmapHeader.yPelsPerMeter = 0;
	bitmapHeader.colorsUsed = 0;
	bitmapHeader.colorsImportant = 0;
	outFile.write( (char *) &fileHeader, sizeof(BitmapFileHeader) );
	outFile.write( (char *) &bitmapHeader, sizeof(BitmapInfoHeader) );
	for(i = 0; i < PASSWORD_SIZE_MAX ; i++)	{
		blockRead_[i] = '\0';
		afterXor_[i] = '\0';
	}
	fseek(inFile,0L,SEEK_SET);
	while(!feof(inFile)) {
		for(i = 0; i < lengthOfPassword_; i++) {
			if(!feof(inFile)) {
				blockRead_[i] = fgetc(inFile);
			}
			else {
				blockRead_[i] = '\0'; // if file ends before block
			}               // use null value
		}
		for(i = 0; i < lengthOfPassword_; i++) {
			afterXor_[i] = previousBlock_[i] ^ blockRead_[i];
			pix.col[count] = afterXor_[i];
			count++;
			if(count >= 3) {
				outFile.write((char *) &pix, sizeof(SinglePixel));
				count = 0;
			}
		}
		previousBlock_[0] = '\0';
		strcpy(previousBlock_, afterXor_);
	}
	fclose(inFile);
	outFile.close();
}
void Encryption::xorDecrypt(char* toDecrypt, char* outputFile, char* password) {
	int i;
	char ch;
	FILE* inFile = fopen(toDecrypt, "rb");
	ofstream outFile(outputFile, ios::binary);
	strcpy(previousBlock_, password);
	lengthOfPassword_ = strlen(password);
	for(i = 0; i < PASSWORD_SIZE_MAX ; i++)	{
		blockRead_[i] = '\0';
		afterXor_[i] = '\0';
	}
	fseek(inFile, 54L, SEEK_SET);
	while(!feof(inFile)) {
		for(i = 0; i < lengthOfPassword_; i++) {
			if(!feof(inFile))
			{
				blockRead_[i] = fgetc(inFile);
				if(blockRead_[i] == '\r') {
					i--;
				}
			}
		}
		for(i = 0; i < lengthOfPassword_; i++) {
			afterXor_[i] = previousBlock_[i] ^ blockRead_[i];
			if(afterXor_[i] != '\0') {
				outFile.put(afterXor_[i]);
			}
		}
		previousBlock_[0] = '\0';
		strcpy(previousBlock_, blockRead_);
	}
	fclose(inFile);
	outFile.close();
}
/****************************************************************************/
/*                                     */
/*   	      A class thats displays the user interface		  */
/*                                     */
/****************************************************************************/
class Gui {
	private:
		int x, y;
	public:
	void drawBox(int left, int top, int right, int bottom);
	void displayTitle();
	void displayQuitMessage();
	void displayEncryptScreen();
	void displayDecryptScreen();
	void displayAboutScreen();
	void displayHelpScreen();
	void displayOptionsScreen();
	void flashLetter(int xPos, int Ypos, char* ch);
};
void Gui::drawBox(int left, int top, int right, int bottom) { // A 3-D box
	setcolor(DARKGRAY);
	line(left, top, right, top);
	line(left, top, left, bottom);
	setcolor(LIGHTGRAY);
	line(left+1, bottom, right, bottom);
	line(right, top+1, right, bottom);
	rectangle(left-2, top-2, right+2, bottom+2);
	rectangle(left+1, top+1, right-1, bottom-1);
	setcolor(WHITE);
	line(left, bottom+1, right, bottom+1);
	line(right+1, top, right+1, bottom+1);
	setfillstyle(1, LIGHTGRAY);
	floodfill(left+5, top+5, LIGHTGRAY);
}
void Gui::flashLetter(int xPos, int yPos, char* ch) {
	setcolor(WHITE);
	outtextxy(xPos, yPos, ch);
	delay(35);
	setcolor(LIGHTGRAY);
	outtextxy(xPos, yPos, ch);
}
void Gui::displayTitle() { // Display the software name
	settextstyle(4, 0, 8);
	setcolor(LIGHTGRAY);
	outtextxy(200, 100, "Gcipher");
	setcolor(DARKGRAY);
	for(x = 200; x < 450; x++) {
		for(y = 100; y < 200; y++) {
			if(getpixel(x, y) == LIGHTGRAY) {
				line(220+(int)((float)x/3.0), 150, x, y);
			}
		}
	}
	setcolor(LIGHTGRAY);
	outtextxy(200, 100, "Gcipher");
	delay(1000);
	flashLetter(200, 100, "G");
	flashLetter(255, 100, "c");
	flashLetter(285, 100, "i");
	flashLetter(305, 100, "p");
	flashLetter(345, 100, "h");
	flashLetter(385, 100, "e");
	flashLetter(415, 100, "r");
	delay(1000);
	cleardevice();
}
void Gui::displayQuitMessage() {    // Giver the user an option to quit or continue
	m.initMouse();
	m.restrict(200, 150, 440, 300);
	setcolor(DARKGRAY);
	rectangle(200, 150, 440, 300);
	setfillstyle(10, DARKGRAY);
	floodfill(201, 151, DARKGRAY);
	setcolor(BLACK);
	rectangle(200, 150, 440, 300);
	setcolor(LIGHTGRAY);
	settextstyle(4, 0, 3);
	setcolor(LIGHTRED);
	outtextxy(240, 180, "Are you sure?");
	m.showMouse();
	while(1) {
		m.getMousePosition(&::bt, &::mx, &::my);
		settextstyle(1, 0, 1);
		setcolor(DARKGRAY);
		outtextxy(250, 230, "Yes");
		outtextxy(350, 230, "No");
		while(::mx > 250 && ::mx < 280 && ::my > 230 && ::my < 250 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(LIGHTRED);
			outtextxy(250, 230, "Yes");
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				m.initMouse();
				exit(0);
			}
		}
		while(::mx > 350 && ::mx < 380 && ::my > 230 && ::my < 250 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(LIGHTRED);
			outtextxy(350, 230, "No");
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				m.hide() ;
				cleardevice();
				m.initMouse();
				m.showMouse() ;
				return;
			}
		}
	}
}
/* A graphical interface that accepts the input and output files, */
/*      the password and produces the output file      */
void Gui::displayEncryptScreen() {
	m.hide() ;
	cleardevice();
	settextstyle(4, 0, 4);
	setcolor(LIGHTRED);
	setfillstyle(1, DARKGRAY);
	floodfill(0, 0, LIGHTGRAY);
	setcolor(BLACK);
	line(5, 455, 635, 455);
	line(5, 455, 5, 475);
	setcolor(LIGHTGRAY);
	line(635, 456, 635, 475);
	line(6, 475, 635, 475);
	setcolor(BLACK);
	outtextxy(243, 33, "Encryption");
	setcolor(LIGHTRED);
	outtextxy(240, 30, "Encryption");
	settextstyle(0, 0, 1);
	setcolor(WHITE);
	outtextxy(60, 120, "Input file name");
	outtextxy(60, 200, "Output file name");
	outtextxy(60, 300, "Password");
	outtextxy(60, 360, "Retype Password");
	drawBox(50, 140, 550, 160);
	drawBox(50, 220, 550, 240);
	drawBox(50, 320, 200, 340);
	drawBox(50, 380, 200, 400);
	m.showMouse() ;
	char toEncrypt[100];  // name of the file to be encrypted
	char outputFile[100]; // Name of the output file
	char password[PASSWORD_SIZE_MAX], passworkCheck[PASSWORD_SIZE_MAX]; // The PASSWORD
	char c = '\0';
	int i = 0;
	for(i = 99; i >= 0; i--) {
		toEncrypt[i] = '\0';
		outputFile[i] = '\0';
	}
	for(i = 0; i < 20; i++)	{
		password[i] = '\0';
		passworkCheck[i] = '\0';
	}
	i = 0;
	while(1) {
		m.showMouse() ;
		m.getMousePosition(&::bt, &::mx, &::my);
	    /* Get the file to be encrypted from the user */
		if(::mx > 50 && ::mx < 550 && ::my > 140 && ::my < 160 && ::bt == 1) {
			while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			m.hide() ;
			x = 8; y = 10;
			c = '\0';
			setcolor(WHITE);
			setfillstyle(1, BLACK);
			m.hide() ;
			bar(51, 141, 549, 159);
			for(i = 99; i >= 0; i--) {
				toEncrypt[i] = '\0';
			}
			i = 0;
			m.showMouse() ;
			gotoxy(x, y);
			while( c != '\r') {
				c = getch();
				gotoxy(x, y);
				if(c != '\b' && c != '\t') {
					toEncrypt[i] = c;
					if(x < 68)
					cout<<c;
					i++;
					x++;
				}
				else
					if(c == '\b' && x > 8) {
						gotoxy(--x, y);
						cout<<" ";
						toEncrypt[i] = '\0';
						i--;
					}
			}
			setfillstyle(1, DARKGRAY);
			bar(6, 456, 634, 474);
			i--;
			toEncrypt[i] = '\0';
		}
		  /* Get the output file from the user */
		if(::mx > 50 && ::mx < 550 && ::my > 220 && ::my < 240 && ::bt == 1) {
			while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			m.hide() ;
			x = 8; y = 15;
			i = 0;
			c = '\0';
			setcolor(WHITE);
			setfillstyle(1, BLACK);
			m.hide() ;
			bar(51, 221, 549, 239);
			for(i = 99;i >= 0;i--) {
				outputFile[i] = '\0';
			}
			i = 0;
			m.showMouse() ;
			while( c != '\r') {
				c = getch();
				gotoxy(x, y);
				if(c != '\b' && c != '\t') {
					outputFile[i] = c;
					if(x < 68)
					cout<<c;
					i++;
					x++;
				}
				else
					if(c == '\b' && x > 8) {
						gotoxy(--x, y);
						cout<<" ";
						outputFile[i] = '\0';
						i--;
					}
			}
			setfillstyle(1, DARKGRAY);
			bar(6, 456, 634, 474);
			i--;
			outputFile[i] = '\0';
		}
			/*  Get the password */
		if(::mx > 50 && ::mx < 200 && ::my > 320 && ::my < 340 && ::bt == 1) {
			while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			m.hide() ;
			for(i = 0;i < PASSWORD_SIZE_MAX;i++)
				password[i] = '\0';
			x = 8; y = 21;
			i = 0;
			c = '\0';
			setcolor(WHITE);
			setfillstyle(1, BLACK);
			m.hide() ;
			bar(51, 321, 199, 339);
			m.showMouse() ;
			while( c != '\r') {
				c = getch();
				gotoxy(x, y);
				if(c != '\b' && c != '\t' && x < 26) {
					password[i] = c;
					if(c != '\r')
					cout<<"*";
					i++;
					x++;
				}
				else
					if(c == '\b' && x > 8) {
						gotoxy(--x, y);
						cout<<" ";
						password[i] = '\0';
						i--;
					}
			}
			setfillstyle(1, DARKGRAY);
			bar(6, 456, 634, 474);
			i--;
			password[i] = '\0';
		}
		  /* Get the password once again */
		if(::mx > 50 && ::mx < 200 && ::my > 380 && ::my < 400 && ::bt == 1) {
			while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			m.hide() ;
			for(i = 0;i < PASSWORD_SIZE_MAX;i++)
				passworkCheck[i] = '\0';
			x = 8; y = 25;
			i = 0;
			c = '\0';
			setcolor(WHITE);
			setfillstyle(1, BLACK);
			m.hide() ;
			bar(51, 381, 199, 399);
			m.showMouse() ;
			while( c != '\r') {
				c = getch();
				gotoxy(x, y);
				if(c != '\b' && c != '\t' && x < 26) {
					passworkCheck[i] = c;
					if(c != '\r')
					cout<<"*";
					i++;
					x++;
				}
				else
					if(c == '\b' && x > 8) {
						gotoxy(--x, y);
						cout<<" ";
						passworkCheck[i] = '\0';
						i--;
					}
			}
			setfillstyle(1, DARKGRAY);
			bar(6, 456, 634, 474);
			i--;
			passworkCheck[i] = '\0';
		}
		/* When user presses the "Encrypt" button */
		while(::mx > 380 && ::mx < 480 && ::my > 290 && ::my < 320 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(RED);
			settextstyle(4, 0, 4);
			outtextxy(380, 280, "Encrypt");
			if(::bt == 1) {
			while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			 /* Give the proper error messages */
				if(toEncrypt[0] == '\0') {
					setcolor(LIGHTGRAY);
					settextstyle(0, 0, 1);
					outtextxy(10, 460, "Please enter the source file");
				}
				else
				if(outputFile[0] == '\0') {
					setcolor(LIGHTGRAY);
					settextstyle(0, 0, 1);
					outtextxy(10, 460, "Please enter the output file");
				}
				else
				if(password[0] == '\0')	{
					setcolor(LIGHTGRAY);
					settextstyle(0, 0, 1);
					outtextxy(10, 460, "Please enter the password");
				}
				else
				if(passworkCheck[0] == '\0') {
					setcolor(LIGHTGRAY);
					settextstyle(0, 0, 1);
					outtextxy(10, 460, "Please retype the password");
				}
				else
				if(strcmp(password, passworkCheck) != 0) {
					setcolor(LIGHTGRAY);
					settextstyle(0, 0, 1);
					outtextxy(10, 460, "Passwords do not match");
				}
				else {
					ofstream outFile;
					ifstream inFile;
					inFile.open(toEncrypt, ios::noreplace);
					outFile.open(outputFile);
					if(!inFile) {
						setcolor(LIGHTGRAY);
						settextstyle(0, 0, 1);
						outtextxy(10, 460, "Invalid input file");
						break;
					}
					else
					if(!outFile) {
						setcolor(LIGHTGRAY);
						settextstyle(0, 0, 1);
						outtextxy(10, 460, "Invalid output file");
						break;
					}
					else {
						inFile.close();
						outFile.close();
						Encryption e;
						e.xorEncrypt(toEncrypt, outputFile, password);
					}
				}
			}
		}
		setcolor(LIGHTGRAY);
		settextstyle(4, 0, 4);
		outtextxy(380, 280, "Encrypt");
	  /* When the user presses the X mark */
	 /* set filenames and passwords to null */
		while(::mx > 620 && ::my < 20 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(LIGHTRED);
			settextstyle(0, 0, 1);
			outtextxy(628, 5, "X");
			setcolor(LIGHTGRAY);
			line(625, 2, 638, 2);
			line(625, 2, 625, 15);
			setcolor(BLACK);
			line(625, 15, 638, 15);
			line(638, 2, 638, 15);
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				for(i = 99;i >= 0;i--) {
					toEncrypt[i] = '\0';
					outputFile[i] = '\0';
				}
				for(i = 0;i < 20;i++) {
					password[i] = '\0';
					passworkCheck[i] = '\0';
				}
				m.hide() ;
				cleardevice();
				m.showMouse() ;
				return;
			}
		}
		setcolor(LIGHTGRAY);
		settextstyle(0, 0, 1);
		outtextxy(628, 5, "X");
		setcolor(DARKGRAY);
		line(625, 2, 638, 2);
		line(625, 2, 625, 15);
		line(625, 15, 638, 15);
		line(638, 2, 638, 15);
	}
}
void Gui::displayDecryptScreen() {
	m.hide() ;
	cleardevice();
	settextstyle(4, 0, 4);
	setcolor(LIGHTRED);
	setfillstyle(1, DARKGRAY);
	floodfill(0, 0, LIGHTGRAY);
	setcolor(BLACK);
	line(5, 455, 635, 455);
	line(5, 455, 5, 475);
	setcolor(LIGHTGRAY);
	line(635, 456, 635, 475);
	line(6, 475, 635, 475);
	setcolor(BLACK);
	outtextxy(243, 33, "Decryption");
	setcolor(LIGHTRED);
	outtextxy(240, 30, "Decryption");
	settextstyle(0, 0, 1);
	setcolor(WHITE);
	outtextxy(60, 120, "Input file name");
	outtextxy(60, 200, "Output file name");
	outtextxy(60, 300, "Password");
	drawBox(50, 140, 550, 160);
	drawBox(50, 220, 550, 240);
	drawBox(50, 320, 200, 340);
	m.showMouse() ;
	char toDecrypt[100];  // name of the file to be encrypted
	char outputFile[100]; // Name of the output file
	char password[PASSWORD_SIZE_MAX]; // The PASSWORD
	char c = '\0';
	int i = 0;
	for(i = 99;i >= 0;i--) {
		toDecrypt[i] = '\0';
		outputFile[i] = '\0';
	}
	for(i = 0;i < 20;i++) {
		password[i] = '\0';
	}
	i = 0;
	while(1) {
		m.showMouse() ;
		m.getMousePosition(&::bt, &::mx, &::my);
	    /* Get the file to be decrypted from the user */
		if(::mx > 50 && ::mx < 550 && ::my > 140 && ::my < 160 && ::bt == 1) {
			while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			m.hide() ;
			x = 8; y = 10;
			c = '\0';
			setcolor(WHITE);
			setfillstyle(1, BLACK);
			m.hide() ;
			bar(51, 141, 549, 159);
			for(i = 99;i >= 0;i--) {
				toDecrypt[i] = '\0';
			}
			i = 0;
			m.showMouse() ;
			gotoxy(x, y);
			while( c != '\r') {
				c = getch();
				gotoxy(x, y);
				if(c != '\b' && c != '\t') {
					toDecrypt[i] = c;
					if(x < 68)
					cout<<c;
					i++;
					x++;
				}
				else
					if(c == '\b' && x > 8) {
						gotoxy(--x, y);
						cout<<" ";
						toDecrypt[i] = '\0';
						i--;
					}
			}
			setfillstyle(1, DARKGRAY);
			bar(6, 456, 634, 474);
			i--;
			toDecrypt[i] = '\0';
		}
		  /* Get the output file from the user */
		if(::mx > 50 && ::mx < 550 && ::my > 220 && ::my < 240 && ::bt == 1) {
			while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			m.hide() ;
			x = 8; y = 15;
			i = 0;
			c = '\0';
			setcolor(WHITE);
			setfillstyle(1, BLACK);
			for(i = 99;i >= 0;i--) {
				outputFile[i] = '\0';
			}
			i = 0;
			m.hide() ;
			bar(51, 221, 549, 239);
			m.showMouse() ;
			while( c != '\r') {
				c = getch();
				gotoxy(x, y);
				if(c != '\b' && c != '\t') {
					outputFile[i] = c;
					if(x < 68)
					cout<<c;
					i++;
					x++;
				}
				else
					if(c == '\b' && x > 8) {
						gotoxy(--x, y);
						cout<<" ";
						outputFile[i] = '\0';
						i--;
					}
			}
			setfillstyle(1, DARKGRAY);
			bar(6, 456, 634, 474);
			i--;
			outputFile[i] = '\0';
		}
			/*  Get the password */
		if(::mx > 50 && ::mx < 200 && ::my > 320 && ::my < 340 && ::bt == 1) {
			while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			m.hide() ;
			for(i = 0;i < PASSWORD_SIZE_MAX;i++)
				password[i] = '\0';
			x = 8; y = 21;
			i = 0;
			c = '\0';
			setcolor(WHITE);
			setfillstyle(1, BLACK);
			m.hide() ;
			bar(51, 321, 199, 339);
			m.showMouse() ;
			while( c != '\r') {
				c = getch();
				gotoxy(x, y);
				if(c != '\b' && c != '\t' && x < 26) {
					password[i] = c;
					if(c != '\r')
					cout<<"*";
					i++;
					x++;
				}
				else
					if(c == '\b' && x > 8) {
						gotoxy(--x, y);
						cout<<" ";
						password[i] = '\0';
						i--;
					}
			}
			setfillstyle(1, DARKGRAY);
			bar(6, 456, 634, 474);
			i--;
			password[i] = '\0';
		}
		/* When user presses the "Decrypt" button */
		while(::mx > 380 && ::mx < 480 && ::my > 290 && ::my < 320 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(RED);
			settextstyle(4, 0, 4);
			outtextxy(380, 280, "Decrypt");
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
			 /* Give the proper error messages */
				if(toDecrypt[0] == '\0') {
					setcolor(LIGHTGRAY);
					settextstyle(0, 0, 1);
					outtextxy(10, 460, "Please enter the source file");
				}
				else
				if(outputFile[0] == '\0') {
					setcolor(LIGHTGRAY);
					settextstyle(0, 0, 1);
					outtextxy(10, 460, "Please enter the output file");
				}
				else
				if(password[0] == '\0')	{
					setcolor(LIGHTGRAY);
					settextstyle(0, 0, 1);
					outtextxy(10, 460, "Please enter the password");
				}
				else {
					ofstream outFile;
					ifstream inFile;
					inFile.open(toDecrypt, ios::noreplace);
					outFile.open(outputFile);
					if(!inFile) {
						setcolor(LIGHTGRAY);
						settextstyle(0, 0, 1);
						outtextxy(10, 460, "Invalid input file");
						break;
					}
					else
					if(!outFile) {
						setcolor(LIGHTGRAY);
						settextstyle(0, 0, 1);
						outtextxy(10, 460, "Invalid output file");
						break;
					}
					else {
						inFile.close();
						outFile.close();
						Encryption e;
						e.xorDecrypt(toDecrypt, outputFile, password);
					}
				}
			}
		}
		setcolor(LIGHTGRAY);
		settextstyle(4, 0, 4);
		outtextxy(380, 280, "Decrypt");
	  /* When the user presses the X mark */
	 /* set filenames and passwords to null */
		while(::mx > 620 && ::my < 20 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(LIGHTRED);
			settextstyle(0, 0, 1);
			outtextxy(628, 5, "X");
			setcolor(LIGHTGRAY);
			line(625, 2, 638, 2);
			line(625, 2, 625, 15);
			setcolor(BLACK);
			line(625, 15, 638, 15);
			line(638, 2, 638, 15);
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				for(i = 99;i >= 0;i--) {
					toDecrypt[i] = '\0';
					outputFile[i] = '\0';
				}
				for(i = 0;i < 20;i++) {
					password[i] = '\0';
				}
				m.hide() ;
				cleardevice();
				m.showMouse() ;
				return;
			}
		}
		setcolor(LIGHTGRAY);
		settextstyle(0, 0, 1);
		outtextxy(628, 5, "X");
		setcolor(DARKGRAY);
		line(625, 2, 638, 2);
		line(625, 2, 625, 15);
		line(625, 15, 638, 15);
		line(638, 2, 638, 15);
	}
}
void Gui::displayAboutScreen() {
	m.hide() ;
	cleardevice();
	setfillstyle(1, DARKGRAY);
	floodfill(0, 0, WHITE);
	settextstyle(4, 0, 4);
	setcolor(LIGHTRED);
	outtextxy(220, 30, "About Gcipher");
	setcolor(LIGHTGRAY);
	settextstyle(11, 0, 1);
	outtextxy(70, 120, "Gcipher version 1 (2006)");
	outtextxy(70, 150, "Graphical Encryption program");
	outtextxy(70, 190, "The user is allowed to use, ");
	outtextxy(70, 210, "modify and redistribute the sourcecode");
	outtextxy(70, 230, "without the author's permission");
	outtextxy(70, 250, "Any sort of comments or suggestions");
	outtextxy(70, 270, "are always welcome.");
	outtextxy(70, 330, "Developed By");
	outtextxy(70, 350, "Somil Bhandari    somil_9@rediffmail.com");
	outtextxy(70, 370, "Swapnil Patil");
	m.showMouse() ;
	while(1) {
		m.getMousePosition(&::bt, &::mx, &::my);
		setcolor(LIGHTGRAY);
		settextstyle(0, 0, 1);
		outtextxy(628, 5, "X");
		setcolor(DARKGRAY);
		line(625, 2, 638, 2);
		line(625, 2, 625, 15);
		line(625, 15, 638, 15);
		line(638, 2, 638, 15);
	  /* When the user presses the X mark */
	 /* set filenames and passwords to null */
		while(::mx > 620 && ::my < 20 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(LIGHTRED);
			settextstyle(0, 0, 1);
			outtextxy(628, 5, "X");
			setcolor(LIGHTGRAY);
			line(625, 2, 638, 2);
			line(625, 2, 625, 15);
			setcolor(BLACK);
			line(625, 15, 638, 15);
			line(638, 2, 638, 15);
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				m.hide() ;
				cleardevice();
				m.showMouse() ;
				return;
			}
		}
	 }
}
void Gui::displayHelpScreen() {
	m.hide() ;
	cleardevice();
	setfillstyle(1, DARKGRAY);
	floodfill(0, 0, WHITE);
	settextstyle(4, 0, 4);
	setcolor(LIGHTRED);
	outtextxy(220, 30, "Help");
	setcolor(LIGHTGRAY);
	settextstyle(11, 0, 1);
	outtextxy(70, 120, "Gcipher version 1 (2006)");
	outtextxy(70, 170, "To encrypt a file, give tis full path in the input file name text box,");
	outtextxy(70, 190, "and give the name of the output file as <filename>.bmp with full path");
	outtextxy(70, 210, "Give a password for the encryption, which will be used as key to ");
	outtextxy(70, 230, "encrypt the file. Remember, the same password is required to decrypt");
	outtextxy(70, 250, "the file. To decrypt the file give the path of the encrypted file and");
	outtextxy(70, 270, "also the password which should be same as the password used to encrypt");
	outtextxy(70, 290, "the file.");
	m.showMouse() ;
	while(1) {
		m.getMousePosition(&::bt, &::mx, &::my);
		setcolor(LIGHTGRAY);
		settextstyle(0, 0, 1);
		outtextxy(628, 5, "X");
		setcolor(DARKGRAY);
		line(625, 2, 638, 2);
		line(625, 2, 625, 15);
		line(625, 15, 638, 15);
		line(638, 2, 638, 15);
	  /* When the user presses the X mark */
	 /* set filenames and passwords to null */
		while(::mx > 620 && ::my < 20 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(LIGHTRED);
			settextstyle(0, 0, 1);
			outtextxy(628, 5, "X");
			setcolor(LIGHTGRAY);
			line(625, 2, 638, 2);
			line(625, 2, 625, 15);
			setcolor(BLACK);
			line(625, 15, 638, 15);
			line(638, 2, 638, 15);
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				m.hide() ;
				cleardevice();
				m.showMouse() ;
				return;
			}
		}
	 }
}
void Gui::displayOptionsScreen() {
	settextstyle(4, 0, 4);
	m.initMouse();
	m.showMouse();
	while(1) {
		settextstyle(4, 0, 4);
		setcolor(DARKGRAY);
		outtextxy(80, 100, "Encrypt");
		outtextxy(80, 150, "Decrypt");
		outtextxy(80, 200, "Help");
		outtextxy(80, 250, "About");
		outtextxy(80, 300, "Quit");
		m.getMousePosition(&::bt, &::mx, &::my);
		/* The Encrypt option */
		while(::mx > 80 && ::mx < 180 && ::my > 110 && ::my < 140 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			settextstyle(4, 0, 4);
			setcolor(LIGHTRED);
			outtextxy(80, 100, "Encrypt");
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				displayEncryptScreen();
			}
		}
		/* The Decrypt option */
		while(::mx > 80 && ::mx < 180 && ::my > 160 && ::my < 190 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			settextstyle(4, 0, 4);
			setcolor(LIGHTRED);
			outtextxy(80, 150, "Decrypt");
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				displayDecryptScreen();
			}
		}
		/* The Help option */
		while(::mx > 80 && ::mx < 180 && ::my > 210 && ::my < 240 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			settextstyle(4, 0, 4);
			setcolor(LIGHTRED);
			outtextxy(80, 200, "Help");
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				displayHelpScreen();
			}
		}
		/* The About option */
		while(::mx > 80 && ::mx < 180 && ::my > 260 && ::my < 290 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			settextstyle(4, 0, 4);
			setcolor(LIGHTRED);
			outtextxy(80, 250, "About");
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				displayAboutScreen();
			}
		}
		/* The quit option */
		while(::mx > 80 && ::mx < 180 && ::my > 310 && ::my < 340 && ::bt == 0) {
			m.getMousePosition(&::bt, &::mx, &::my);
			setcolor(LIGHTRED);
			settextstyle(4, 0, 4);
			outtextxy(80, 300, "Quit");
			if(::bt == 1) {
				while(::bt == 1) m.getMousePosition(&::bt, &::mx, &::my);
				m.hide();
				displayQuitMessage();
			}
		}
	}
}
void main() {
	int graphDetect = DETECT, graphMode;
	initgraph(&graphDetect, &graphMode, "");
	Gui gui;
	gui.displayTitle();  /* Disply the title screen */
	gui.displayOptionsScreen(); /* Display the options */
}
```

