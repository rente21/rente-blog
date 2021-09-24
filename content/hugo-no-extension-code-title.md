---
title: 'Hugoでコードブロックにタイトルをつける方法（全拡張子対応）'
date: 2021-09-24
tags: ['Hugo']
---

hugo のコードブロックにタイトルをつける方法を解説している記事があった。  
[Hugo のコードブロックに Qiita のような Title をつける | AABrain](https://aakira.app/blog/2018/12/code-block-title/)

しかしこの記事の内容だと、「hoge.go」「hoge.php」などの拡張子のあるファイルしか対応できなかった。

そこで.htaccess などの**拡張子がないファイルも含め、全て対応できる方法**を紹介する。

## Markdown の Attribute 機能を使用する {nuxt="cnkcnkd"}

Markdown には Attribute という属性を付与する機能がある。  
例えば、このようにすると

```markdown
# hello world {hoge="text"}
```

```html
<h1 hoge="text">hello world</h1>
```

が描画される。

これを応用し、記事の markdown ファイルでこのような表記をしてみる。(fn は filename を表している)  
**属性はダブルクォーテーションで囲まないと描画されないので注意だ。**

````markdown
```ApacheConf {fn=".htaccess"} <!-- ダブルクォーテーションで囲むこと！！ -->
Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.html [QSA,L]
```
````

するとこんな html が生成される。

```html
<div class="highlight" fn=".htaccess">
  <pre
    tabindex="0"
    style="color:#f8f8f2;background-color:#272822;-moz-tab-size:4;-o-tab-size:4;tab-size:4"
  >
    <code class="language-ApacheConf" data-lang="ApacheConf">
      Options -MultiViews
      RewriteEngine<span style="color:#66d9ef">On</span>
      RewriteCond %{REQUEST_FILENAME} !-f
      RewriteRule ^ index.html [QSA,L]
    </code>
  </pre>
</div>
```

ここで fn 属性をもとに、DOM を生成する js を書く。

```js
const list = document.querySelectorAll('.highlight'); // highlightクラスを全て取得
for (i = 0; i <= list.length - 1; i++) {
  const fn = list[i].getAttribute('fn'); // ファイル名
  const code = list[i].firstElementChild.firstElementChild;
  if (fn) {
    const div = document.createElement('div');
    div.textContent = fn;
    div.classList.add('code-name');
    code.parentNode.insertBefore(div, code);
  }
}
```

するとこんな html が生成される。

```html
<div class="highlight" fn=".htaccess">
  <pre
    tabindex="0"
    style="color:#f8f8f2;background-color:#272822;-moz-tab-size:4;-o-tab-size:4;tab-size:4"
  >
    <!-- 追加された -->
    <div class="code-name">.htaccess</div> 
    <!---->
    <code class="language-ApacheConf" data-lang="ApacheConf">
      Options -MultiViews
      RewriteEngine<span style="color:#66d9ef">On</span>
      RewriteCond %{REQUEST_FILENAME} !-f
      RewriteRule ^ index.html [QSA,L]
    </code>
  </pre>
</div>
```

こんな感じで適当に css を付与してあげれば...

```css
pre code {
  display: block;
  font-size: 16px;
  padding: 12px;
  overflow: scroll;
}

/* 重要な部分 */
.code-name {
  display: inline-block;
  position: relative;
  padding: 4px 8px;
  background-color: #e7e9eb;
  color: #485a60;
  font-size: 13px;
  font-weight: 400;
}
```

こんな感じで描画される。

```ApacheConf {fn=".htaccess"}
Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.html [QSA,L]
```

ちなみに markdown ファイルの

````markdown
```ApacheConf {fn=".htaccess"}

```
````

の ApacheConf の部分は決められた言語名を書くのだが、以下の url に対応言語とその時にどのように書けば良いかが書かれている。  
https://gohugo.io/content-management/syntax-highlighting/#list-of-chroma-highlighting-languages
