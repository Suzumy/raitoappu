# fake
## Wanictf 2021 spring
## Web
***
問題としてURLが渡されるため、まずはそこにアクセスする。  
すると、色どり豊かなリンクボタンが大量に書かれたページが出る。  
しかもどのリンクを押してもどこにも飛ばない。どうやらこの中から本物を見つける問題っぽい。  
一つ一つ押していく手もあるが、今回はソースコードを確認してみる。  
ソースコードを確認すると割とすぐに以下のようなコードが見つかる
```
    <a href="144c9defac04969c7bfad8efaa8ea194.html" style="display: none;">
      <button type="button" class="btn btn-primary">Link</button>
    </a>
```
よく見るとdisplay: noneとなっているため、さっきの画面には表示されていなかったぽい。  
とりあえずリンクが見つかったため、ここにアクセスすると、フラグが出てくるため、これをsubmitする。