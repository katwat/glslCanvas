# これは何？

[GlslCanvas](https://github.com/patriciogonzalezvivo/glslCanvas) 0.1.7 を fork して、WebGL2 および GLSL ES 3.0 にも対応可能とした版です。  
基本的な使い方はオリジナルの[GlslCanvas](https://github.com/patriciogonzalezvivo/glslCanvas)を参照してください。

## canvasに指定可能なdata属性を追加

対象 canvas に ```data-es3="true"``` という属性と値を追加すると、WebGL2 および GLSL ES 3.0 対応となります。ちなみに値は空文字で無ければ何でも構いません。

また、シェーダーのコードの先頭に ```#version 300 es``` を自動で追加するようになっているので、別途記述する必要はありません。

なお、GLSL ES 1.0 とは記述の仕方が異なる部分があるので注意してください。[こちら](https://wgld.org/d/webgl2/w003.html)が参考になります。

## uniform 変数を追加

[こちら](https://webglfundamentals.org/webgl/lessons/webgl-shadertoy.html)を参考に、[Shadertoy](https://www.shadertoy.com/)で導入されている uniform 変数をいくつか追加しました。

* ```int u_frame``` : 開始からのフレーム番号（0～）
* ```float u_frameRate``` : 秒あたりの描画フレーム数（いわゆるFPS）

## テクスチャの自動ロード機能

オリジナルの[GlslCanvas](https://github.com/patriciogonzalezvivo/glslCanvas)では説明されていませんが、以下のようにシェーダのコードにコメントでファイル名を書いておくと、自動でテクスチャを読み込んで uniform 変数に設定される機能があります。便利です。

```uniform sampler2D u_logo; // data/logo.jpg```

## オフスクリーン描画バッファ機能

これまたオリジナルの[GlslCanvas](https://github.com/patriciogonzalezvivo/glslCanvas)では説明されていませんが、__#ifdef__ や __#if defined()__ のプリプロセッサを以下のようにシェーダのコードに書くと、バッファが自動で作られ、それらを対象とした描画が行える機能があります。

```glsl
	︙
uniform sampler2D u_buffer0;
uniform sampler2D u_buffer1;
	︙
#if defined( BUFFER_0 )
	︙
(u_buffer0を対象としたコード）
	︙
#elif defined( BUFFER_1 )
	︙
（u_buffer1を対象としたコード）
	︙
#else
	︙
（メインバッファ=canvasを対象としたコード）
	︙
#endif
	︙
```

上記のように書いた場合、バッファ０への描画、バッファ１への描画、メインバッファへの描画の計３回の描画が一回のレンダー処理で行われます。バッファはテクスチャとして参照できますので、うまく使うことで色々な効果を表現できたりするようです。[こちら](https://github.com/patriciogonzalezvivo/glslCanvas/blob/master/buffers.html)にサンプルコードがあります。

## Shadertoyのコードを移植してみるには

[Shadertoy](https://www.shadertoy.com/)には様々なコードが紹介されていますが、それらを比較的容易に移植することが可能です。

以下にやり方の一例を挙げます。ただし、あくまで一例です。調整が必要になる場合があるかもしれません。また、うまく行かない場合もあるかもしれません。ご了承ください。

```html
<canvas id="glslCanvas" width="320" height="180" data-es3="true" data-fragment="
#ifdef GL_ES
precision mediump float;
#endif

// 必要な uniform を列挙。
uniform vec2 u_resolution;
uniform float u_time;
uniform int u_frame;
uniform sampler2D u_buffer0;

// Shadertoyでの名前を変換。
#define iResolution u_resolution
#define iTime u_time
#define iFrame u_frame
#define iChannel0 u_buffer0

#if defined( BUFFER_0 )
	︙
（ここに Shadertoy の「Buffer A」のコードを書く）
	︙
#else
	︙
（ここに Shadertoy の「Image」のコードを書く）
	︙
#endif

out vec4 fragColor;
void main() {
	mainImage(fragColor,gl_FragCoord.xy);
}
"></canvas>
```
