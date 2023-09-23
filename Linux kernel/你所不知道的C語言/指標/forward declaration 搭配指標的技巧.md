
oltk.h
```
struct oltk; // 宣告 (incomplete type, void)
struct oltk_button;
typedef void oltk_button_cb_click(struct oltk_button *button, void *data);
typedef void oltk_button_cb_draw(struct oltk_button *button,
                                 struct oltk_rectangle *rect, void *data);
struct oltk_button *oltk_button_add(struct oltk *oltk,
                                    int x, int y,
                                    int width, int height);
```

`struct oltk` 和 `struct oltk_button` 沒有具體的定義 (definition) 或實作 (implementation)，僅有宣告 (declaration)

oltk.c
```
struct oltk {
  struct gr *gr;
  struct oltk_button **zbuttons;
  ...
  struct oltk_rectangle invalid_rect;
};
```


- `oltk.h` 只包含 interface
	- 已公開的函式如 `oltk_button_add` 都是透過 pointer 存取給定的位址，而不用顧慮具體 struct oltk 的實作
	- 可以隱藏實作細節，還能兼顧二進位的相容性 (binary compatibility)
- 同理，struct oltk_button 不管怎麼變更，在 struct oltk 裡面也是用給定的 pointer 去存取，保留未來實作的彈性。

#C語言 #Pointer 