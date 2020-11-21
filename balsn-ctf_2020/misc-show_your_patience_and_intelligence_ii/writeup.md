## Misc / 380 - Show your Patience and Intelligence II

### Solution
By @raagi 

依據題目提示可以知道 LED 的控制器型號為 MAX7219 ，而根據製造商的 datasheet 可以知道 MAX7219 的輸入端分別有三種來源 `SCLK (CLK)` 、 `MOSI(DIN)` 、 `CS(LOAD)` ，對照這三者的結果產生輸出的 `DOUT` 跟 `SEG` 會組成能代表目前模式與定位某個 LED 明滅與否的 16-bit 指令，接腳如下圖。

![](https://i.imgur.com/DGrfPkN.png)

再來從 datasheet 的 table 2 得知， 16-bit 的指令可分成兩個部分，高八位代表 address 跟 mode ，低八位則是代表該行的哪些燈該亮的 data

| D15-D12 | D11-D8 | D7-D4 | D3-D0 |
| ------- | ------ | -------- | --- |
| Don't care | address | data | data |

其中高八位的前四位是 don't care bit ，因此只需要注意後四位，如果是 0x1 ~ 0x8 之間，則代表此次送出的資料是第 0x1 ~ 0x8 欄的資訊，0x9 ~ 0xF 則是類似 debug mode ，不會在 LED 燈上有輸出但能對控制器內部做設定的更改，而 0x0 則代表 no-op ，在串連多個 LED 燈時，若接收到 no-op 訊號就保持上一次的狀態不做更動，可以讓每個串連的 LED 之間彼此不受影響。

接著看到輸入的三個訊號如何決定輸出， `CLK` 代表 high-triggered 的 clock signal 所以每當它從 0 變成 1 的瞬間，當下的資料就會被採用；而 `CS` 則是 low triggered ，每次從 1 變成 0 時開始接收資料到暫存器， 0 變成 1 時停止接收資料。示意圖如下：

![](https://i.imgur.com/6m51IiL.png)

而 `[D7-D0]` 低八位分別代表了由下往上、單一欄中的每一個 LED 燈，如果是 1 就亮起來 0 就不亮。

從以上的資訊可以得知，假如有一筆資料為 `0000 0001 1110 0100` 那麼 LED 燈的狀態會長這樣：

![](https://i.imgur.com/KLx2V2J.png)

於是，我們就可以開始解了。題目有給一個 vcd dump ，把它丟到 gtkWave 中可以看到四個 signal 的示波圖：

![](https://i.imgur.com/KF3Ef3l.png)

可以看得出來 Channel_0 代表 `CS` ， Channel_1 代表 `CLK` ， `Channel_8` 代表 `DIN`。但因為 vcd file 包含了時序的資料，不方便操作，我在網路上找到了一個工具叫做 vcdcat ，使用它 parse 出來的資料來寫 script。

```python=
for x in range(len(matrix)):
	mode = matrix[x][4]*8 + matrix[x][5]*4 + matrix[x][6]*2 + matrix[x][7]*1
	if x < len(matrix) - 4:
		next_mode = matrix[x+1][4]*8 + matrix[x+1][5]*4 + matrix[x+1][6]*2 + matrix[x+1][7]*1
		if next_mode < 9 and next_mode > 0 and next_mode < mode:
			print("")
	if mode == 9:
		print("decode")
	elif mode == 10:
		print("intensity")
	elif mode == 11:
		print("scan limit")
	elif mode == 12:
		print("shut down")
	elif mode == 15:
		print("test")
	elif mode == 0:
		continue
	else:
		# print(mode, end='')
		for i in matrix[x][8:16]:
			print('X' if i ==1 else ' ', end=' ')
		print()
```
印出來長這樣，記得頭往右轉看：

![](https://i.imgur.com/Ph1WX7x.png)

Flag: `BALSN{I_spent_a_lot_of_time_drawing_letters_QAQ}`

### Reference
* [使用 MAX7219 驅動七段顯示器 - MediaTek Lab](https://docs.labs.mediatek.com/resource/linkit7697-arduino/zh_tw/tutorial/driving-7-segment-displays-with-max7219)
* [MAX7219/MAX7221 Datasheet](https://datasheets.maximintegrated.com/en/ds/MAX7219-MAX7221.pdf)
* [ cirosantilli / vcdvcd - GitHub](https://github.com/cirosantilli/vcdvcd)

