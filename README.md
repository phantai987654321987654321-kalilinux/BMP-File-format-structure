# Written By Michael Phan
# BMP file format structure
# Định nghĩa
- Là tệp ảnh Bitmap Image
- Định dạng này có thể được đọc đa nền tảng(Microsoft, Linux,Mac) bởi có tính chất:
	- Independent of graphics adapter(độc lập với card đồ hoạ)
	- hoặc gọi là device independent bitmap file format(DIB)-Định dạng file độc lập với các thiết bị
- Lưu trữ data ở dạng ảnh 2 chiều cả chế độ đơn sắc(monochrome) và đa sắc(colour) với đa dạng colour depth(bits depth)-là độ dài bit biểu diễn cho các màu trên 1 điểm ảnh
- Do sự phát triển liên tục của định dạng tệp này, cấu trúc .bmp có thể khác nhau tùy theo phiên bản Bitmap nhưng luôn tuân thủ như sau
	1. Bitmap file header: luôn 14 bytes
	2. DIB header: mang thông tin tổng thể của file và version của bmp(số byte có thể là 12/40/108/124 và thường là 40, xác định chữ ký theo chuẩn Little Endian)
	3. Colour Palette: tuỳ chọn colour depth
	4. Pixel data(Pixel array, BGR rows + padding): Dữ liệu ảnh

## 1. Bitmap file header
- Bắt đầu tại địa chỉ **0x00(Hex)**, bao gồm **14 byte**, chi tiết được thể hiện như sau:

Offset (Hex) |	Kích thước  |	Định nghĩa (C/C++)   |	Kiểu dữ liệu  |	Phân tích chức năng
-------------|--------------|------------------------|----------------|-------------------------------------------------
0x00	     |	2 bytes	    |	WORD (bfType)	     |	uint16_t      |	Chữ ký định dạng: Bắt buộc mang mã hex 0x4D 0x42 (tương ứng mã ASCII ký tự B và M).
0x02	     |	4 bytes	    |	DWORD (bfSize)	     |	uint32_t      |	Tổng kích thước: Kích thước của toàn bộ tập tin BMP tính bằng byte; giá trị này tuân theo chuẩn Little Endian.
0x06         |	2 bytes	    |	WORD (bfReserved1)   |	uint16_t      |	Dữ liệu sao lưu 1: Bắt buộc là 0x00 0x00.
0x08	     |  2 bytes	    |	WORD (bfReserved2)   |	uint16_t      |	Dữ liệu sao lưu 2: Bắt buộc là 0x00 0x00.
0x0A	     |	4 bytes	    |	DWORD (bfOffBits)    |	uint32_t      |	Con trỏ Offset: Khai báo vị trí byte chính xác bắt đầu vùng dữ liệu Pixel (thường là byte thứ 54 đối với ảnh 24-bit).

## 2. DIB Header
- Khác biệt cốt lõi giữa các phiên bản BMP hiện đại nằm ở kích thước của DIB Header. Hệ thống đọc giá trị DWORD (4 bytes-unit32_t)
	- tại offset 0x0E để xác định kích thước khối này, từ đó xác định chính xác nó đang xử lý phiên bản BMP nào.

### 2.1. BMP Version 3 (bao gồm **40byte(bfSize)**, chữ ký ở Offset 0x0E là **0x28 0x00 0x00 0x00**)
Offset (Hex) |	Kích thước  |	Định nghĩa (C/C++)   |	Kiểu dữ liệu  |	Phân tích chức năng
-------------|--------------|------------------------|----------------|------------------------------------------------
0x12	     |	4 bytes	    |	LONG (biWidth)	     |	int32_t       |	Chiều rộng vật lý của ảnh tính bằng pixel(bắt buộc giá trị dương)
0x16	     |	4 bytes	    |	LONG (biHeight)      |	int32_t       |	Chiều dài vật lý của ảnh tính bằng pixel(giá trị dương hoặc âm sẽ quét Top to Bottom)
0x1A         |	2 bytes	    |	WORD (biPlanes)      |	uint16_t      |	Số mặt phẳng màu(luôn là 1: 0x01 0x00)
0x1C	     |  2 bytes	    |	WORD (biBitCounts)   |	uint16_t      |	ColourDepth: số lượng bit cấu thành 1 pixel(giá trị 1/4/8/16/24/32).
0x1E	     |	4 bytes	    |	DWORD (biCompression)|	int32_t       |	'0' = BI_RGB: ko nén, phổ biến nhất<br>'1' = BI_RLEB8: nén RLE cho ảnh 8 bit<br>'2' = BI_RLEB4: nén RLE cho ảnh 4 bit<br>'3' = BI_BITFIELDS: dùng bitmask
0x22	     |	4 bytes	    |	DWORD (biSizeImage)  |	uint32_t      |	Dung lượng ảnh thô: -Kích thước vùng dữ liệu thô(Pixel Array+padding); nếu biCompression = '0' thì có thể để giá trị này là 0
0x26         |	4 bytes	    |	LONG (biXPelsPerMeter)|	uint16_t      |	Pixel pers meter along X-axis<br>'0' = Default<br>'2835' = means 72 DPI-Dots per inch
0x2A	     |  4 bytes	    |	LONG (biYPelsPerMeter)|	uint16_t      |	Pixel pers meter along Y-axis<br>'0' = Default<br>'2835' = means 72 DPI-Dots per inch
0x2E	     |	4 bytes	    |	LONG (biClrUsed)     |	int32_t       |	Chỉ định số lượng chỉ mục màu trong bảng màu(Color palete) thực sự được bitmap sử dụng<br>'0' = Sử dụng con trỏ để lấy giá trị màu và có thể lấy được tối đa các màu trong bảng màu theo giá trị biBitCounts, Color palete = 4 * 2^biBitCounts<br>(biClrUsed =! '0' || '0' < biBitCounts <'16') = Tương tự ở trên nhưng không lấy tối đa nếu biBitCounts < '8'<br>(biClrUsed =! '0' ||'8' < biBitCounts <= '32') = Lấy kích thước màu trực tiếp, nếu biBitCounts ='16' hoặc '32', nó sẽ cần 3 DWORD bitmask sau 40byte DIB header
0x32         |	4 bytes	    |	DWORD (biClrImportant)|	uint16_t      |	Chỉ định số lượng chỉ mục màu quan trọng được hiển thị trên bitmap<br>'0' = Tất cả đều quan trọng

- bfOffBits = 14(File Header) + (DIB Header) + biClrUsed*4

- Surface Stride: Khi biCompression = '0', Stride được định nghĩa là số lượng byte cần thiết để biểu diễn một hàng pixel

```cpp	
	<C++>
		stride = ((((biWidth * biBitCount) + 31) & ~31) >> 3);
		biSizeImage = abs(biHeight) * stride;
	</>
```

- Bitmask structure: các giá trị này phải liền kề nhau và **không được chồng chéo**
```text
	<Pseudocode>
	typedef _WinNtBitfieldsMasks
	{
		DWORD RedMask;         /* Mask identifying bits of red component */
		DWORD GreenMask;       /* Mask identifying bits of green component */
		DWORD BlueMask;        /* Mask identifying bits of blue component */
	} WINNTBITFIELDSMASKS;
	+16-bit:
		DWORD RedMask   = 0xF8000000;     /* 1111 1000 0000 0000 0000 0000 0000 0000 */
		DWORD GreenMask = 0x07E00000;     /* 0000 0111 1110 0000 0000 0000 0000 0000 */
		DWORD BlueMask  = 0x001F0000;     /* 0000 0000 0001 1111 0000 0000 0000 0000 */
	+32-bit:
		DWORD RedMask   = 0xFFC00000;     /* 1111 1111 1100 0000 0000 0000 0000 0000 */
		DWORD GreenMask = 0x003FF000;     /* 0000 0000 0011 1111 1111 0000 0000 0000 */
		DWORD BlueMask  = 0x00000FFC;     /* 0000 0000 0000 0000 0000 1111 1111 1100 */
	</>
```

### 2.2. BMP Version 4 (bao gồm **108byte(bfSize)**, chữ ký ở Offset 0x0E là **0x6C 0x00 0x00 0x00**)
- Ver 4 kế thừa hoàn toàn các tính năng của Ver 3 và thêm vào 9 trường(fields) khác trong đó có `CIEXYZTRIPLE` `bV4Endpoints` bao gồm 9 thành phần nhỏ hơn
- Đối với `bV4Compression`, nó thêm vào 2 giá trị khác:
	+ '4' = `Bi_JPEG`: Định dạng nén này là giải pháp giảm băng thông khi truyền ảnh JPEG, cho dữ liệu thô được "bao gói" bởi định dạng .bmp
	+ '5' = `Bi_PNG`: Tương tự
>**Do đó** khi `bV4Compression` mang 2 giá trị trên, `bV4SizeImage` cũng sẽ thay đổi thành kích thước của tệp JPEG hoặc PNG
- Tương tự Ver3, Ver4 cũng sử dụng bitmask khi giá trị `bV4Compression` = '3', đặc biệt thêm `AlphaMask` thể hiện độ trong suốt(transparent) của ảnh và tất cả đều là kiểu dữ liệu uint32_t
	+16-bit:
		DWORD RedMask   = 0x07C00000;     /* 0000 0111 1100 0000 0000 0000 0000 0000 */
		DWORD GreenMask = 0x003E0000;     /* 0000 0000 0011 1110 0000 0000 0000 0000 */
		DWORD BlueMask  = 0x0001F000;     /* 0000 0000 0000 0001 1111 0000 0000 0000 */
		DWORD AlphaMask = 0xF8000000;     /* 1111 1000 0000 0000 0000 0000 0000 0000 */
	+32-bit:
		DWORD RedMask   = 0x00FF0000;     /* 0000 0000 1111 1111 0000 0000 0000 0000 */
		DWORD GreenMask = 0x0000FF00;     /* 0000 0000 0000 0000 1111 1111 0000 0000 */
		DWORD BlueMask  = 0x000000FF;     /* 0000 0000 0000 0000 0000 0000 1111 1111 */
		DWORD AlphaMask = 0xFF000000;     /* 1111 1111 0000 0000 0000 0000 0000 0000 */
- DWORD `bV4CSType` uint32_t 4byte: Không gian màu của DIB bắt buộc mang giá trị `LCS_CALIBRATED_RGB`(LogicalColorSpace Enumeration Clibrated RGB) = '0'. Trường này để đưa ra một cấu trúc linh động, giúp hiệu chỉnh(calibrate) màu sắc
				- theo thiết bị sử dụng không gian màu khác với không gian màu tạo ra dữ liệu gốc để tránh gây xung đột hoặc lỗi ảnh.
- Long `bV4Endpoints` 32-bit fixed point value 36byte: Cấu trúc `CIEXYZTRIPLE` định nghĩa cấu trúc sắc độ(chromaticity) theo chuẩn CIE(Commission Internationale de l'Eclairage - An international Commission on Illumination in Vienna, Austria)
			- cho điểm cuối(endpoints) 3 màu đỏ, lục, xanh. Nếu `CSType != LCS_CALIBRATED_RGB` thì `bV4Endpoints` bị bỏ qua.
-DWORD `bV4GammaRed/Green/Blue` 32-bit fixed point value 12byte: kiểu dữ liệu 32bit với dấu chấm tĩnh(32-bit fixed point value), xác định cường độ màu theo đường cong màu(toned response curve)

### 2.3. BMP Version 5 (bao gồm **124byte(bfSize)**, chữ ký ở Offset 0x0E là **0x7C 0x00 0x00 0x00**)
- DWORD `bV4CSType` uint32_t 4byte: Không gian màu của DIB có 5 giá trị gán, nếu giá trị  là `PROFILE_LINKED` hoặc `PROFILE_EMBEDDED` thì gamma và endpoint bị bỏ qua:
	+ LCS_CALIBRATED_RGB
	+ LCS_sRGB = 0x73524742 : xác đinh giá trị màu tuân theo không gian màu sRBG chuẩn
	+ LCS_WINDOWS_COLOR_SPACE = 0x57696E20 : xác đinh giá trị màu tuân theo không gian màu mặc định sRGB của hệ thống đọc ảnh
	+ PROFILE_LINKED = 0x4C494E4B : Nó khai báo việc nhận dạng sẽ thông qua trường bV5ProfileData, và trường này sẽ trỏ vào một file riêng biệt khác chứa link dẫn tới không gian màu gốc của hệ thống
	+ PROFILE_EMBEDDED = 0x4D424544 : "This value indicates that `bV5ProfileData` points to a memory buffer that contains the profile to be used"
				>Giải thích: Một không gian màu tuân theo ICC(International Color Consortium) được nhúng vào cuối file. Khi thực thi file .bmp, phần nhúng này sẽ được ghi vào bộ nhớ đệm(memory buffer), khi thực thi file sẽ được trỏ tới vùng nhớ đệm đang đọc phần nhúng
- DWORD `bV5Intent` uint32_t 4byte: là trường quy định quy tắc cách dải màu gamut được ánh xạ vào:
	Value	     		|	Intent	    |	    ICC name	     |	Meaning
--------------------------------|-------------------|------------------------|------------------------------
 LCS_GM_ABS_COLORIMETRIC = '8'	|	Match	    |	Absolute Colorimetric|	Cố đinh các điểm trắng(white point) và phối màu với các màu gần đó nhất khi thực hiện trong dải màu gamut của thiết bị đọc ảnh
 LCS_GM_BUSINESS         = '1'	|	Graphic	    |	Saturation	     |	Giữ nguyên độ bão hoà(Saturation), dùng cho bảng biểu và một vài ngữ cảnh khác có liên quan đến tông màu phẳng (undithered colors)-tức là thuần 1 màu 
 LCS_GM_GRAPHICS	 = '2' 	|  	Proof       |	Relative Colorimetric|	Đảm bảo màu sắc được ánh xạ tuyệt đối theo dải màu gamut dựa vào giá trị đo lường màu sắc (Colorimetric), dùng cho thiết kế đồ hoạ và bảng màu định danh
 LCS_GM_IMAGES	     	 = '4'	|	Picture	    |	Perceptual	     |	Giữ nguyên độ tương phản(Contrast), sử dụng cho ảnh chụp và hình ảnh tự nhiên
- DWORD bV5ProfileData uint32_t 4byte: Nó ghi nhận giá trị là số byte thể hiện khoảng cách tính từ `BITMAPV5HEADER`(0x0E) cho tới con trỏ lưu vị trí bắt đầu của khối dữ liệu nhúng ICC.
					- Nếu là dạng `PROFILE_LINKED`, nó sẽ đọc cho tới vị trí thể hiện NULL(0x00) thể hiện kết thúc đường dẫn Link
					- Cần lưu ý là nó không được Encode bằng Unicode mà phải sử dụng bộ ký tự của Windows 
- DWORD `bV5ProfileSize` uint32_t 4byte: Kích thước của phần nhúng tính bằng byte
- DWORD `bV5Reserved` uint32_t 4byte: Đặt giá trị là '0'

## 3. Pixel Array
- Khi đọc hết số byte của header và DIB, trình đọc ảnh sẽ tiến hành đọc từ cuối file(từ phải sang trái từ dưới lên trên) cho tới vị trí offset thể hiện trong `bfOffBits`
- Cách đọc file từ cuối lên là tuân theo **Little Endian**, tức đọc theo **Least significant bit** nhưng nó lại thể hiện cách pixel sắp xếp trong ảnh thì khác:
	- Hiện pixel từ phải sang trái từ trên xuống dưới trên ảnh
- Ví dụ: Cho 1 ảnh với số lượng pixel là 6x6, một màu sẽ được biểu diễn bởi số lượng pixel 4x4
```text
Offset|	01	23	45
_______
01	đỏ	xanh	cam

23	lục	đen	tím

45	đỏ	trắng	chàm

</Hex>
Offset							      bfOffBits
____________________________________________________________________________________________________________________________________________								
000030 |	 00	 00	 00	 00	 00	 00	|00	 00	 ff|	[00	 00	 ff]	|ff	 ff	 ff|	[ff

000040 |	 ff	 ff]	|e7	 bf	 c8|	[e7	 bf	 c8]	|00	 00|	[00	 00	 ff]	|00	 00	 ff|

000050 |	[ff	 ff	 ff]	|ff	 ff	 ff|	[e7	 bf	 c8]	|e7	 bf	 c8|	[00	 00]	|00	 ff

000060 |	 00|	[00	 ff	 00]	|00	 00	 00|	[00	 00	 00]	|ff	 00	 80|	[ff	 00	 80]

000070 |	|00	 00|	[00 	 ff	 00]	|00	 ff	 00|	[00	 00	 00]	|00	 00	 00|	[ff	 00

000080 |	 80]	|ff	 00	 80|	[00	 00]	|00	 00	 ff|	[00	 00	 ff]	|ff	 80	 00|	[ff

000090 |	 80	 00]	|00	 80	 ff|	[00	 80	 ff]	|00 	 00|	[00	 00	 ff]	|00	 00	 ff|

0000a0 |	[ff	 80	 00]	|ff	 80	 00|	[00	 80	 ff]	|00	 80	 ff|	[00	 00]
```

- [] or | |: là một cụm chỉ thị màu gồm 6byte

- đỏ	0x0000ff
- xanh	0xff8000
- cam	0x0080ff
- lục	0x00ff00
- đen	0x000000
- tím	0xff0080
- trắng	0xffffff
- chàm	0xe7bfc8


# REFERENCES
>https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-wmf/4813e7fd-52d0-4f42-965f-228c8b7488d2
>https://learn.microsoft.com/en-us/windows/win32/api/wingdi/ns-wingdi-bitmapinfoheader
>https://jacobfilipp.com/DrDobbs/articles/DDJ/1994/9409/9409a/9409a.htm
>https://jacobfilipp.com/DrDobbs/articles/DDJ/1995/9504/9504c/9504c.htm
>https://engineering.purdue.edu/ece264/19sp/hw/HW11
>https://en.lntwww.de/index.php?title=Digital_Signal_Transmission/Applications_for_Multimedia_Files&direction=prev&oldid=49491
>https://docs.fileformat.com/image/bmp/
>https://www.leadtools.com/help/leadtools/v19merged/dh/multimedia/mf/leadtools.mediafoundation~leadtools.mediafoundation.bitmapinfoheader_members.html
>https://upload.wikimedia.org/wikipedia/commons/7/75/BMPfileFormat.svg
