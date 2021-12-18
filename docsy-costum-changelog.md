# Docsy Custom Changes

---

## [Shortcodes] bilibili 视频插入

修改 `assets/scss/main.scss`

```scss
// [Shortcodes] bilibili 视频插入
#bilibiliplayer {
    width: 60%;
    height: 500px;
}
@media only screen and (min-device-width: 320px) and (max-device-width: 480px) {
    #bilibiliplayer {
    width: 100%;
    height: 250px;
    }
}
```

增加 `layouts/shortcodes/bilibili.html`

```html
{{ $videoID := index .Params 0 }}
{{ $pageNum := index .Params 1 | default 1}}

{{ if (findRE "^[bB][vV][0-9a-zA-Z]+$" $videoID) }}
<iframe id="bilibiliplayer" src="//player.bilibili.com/player.html?bvid={{ $videoID }}&page={{ $pageNum }}" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" loading="lazy"></iframe>
{{ else }}
<iframe id="bilibiliplayer" src="//player.bilibili.com/player.html?aid={{ $videoID }}&page={{ $pageNum }}" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" loading="lazy"></iframe>
{{ end }}
```

用法：`{{< bilibili bvid page >}}` 其中 BVID 是 bilibili 视频的 bv 号（以 BV 开头），page 是视频分页（默认为 1）；
用法：`{{< bilibili aid page >}}` 其中 aid 是 bilibili 视频的 av 号（数字），page 是视频分页（默认为 1）。

## [Shortcodes] asciinema 命令行录制插入

增加 `layouts/shortcodes/asciinema.html`

```html
{{ $id := .Get 0 }}
<script type="text/javascript" src="https://asciinema.org/a/{{ $id }}.js" id="asciicast-{{ $id }}" data-rows="10" async></script>
```

用法：`{{< asciinema id >}}`

## [Shortcodes] 选项卡

增加 `layouts/shortcodes/tabfolder.html`

```
{{- .Page.Scratch.Add "tabset-counter" 1 -}}
{{- $tab_set_id := .Get "name" | default (printf "tabset-%s-%d" (.Page.RelPermalink) (.Page.Scratch.Get "tabset-counter") ) | anchorize -}}
{{- $tabs := .Scratch.Get "tabs" -}}
{{- if .Inner -}}{{- /* We don't use the inner content, but Hugo will complain if we don't reference it. */ -}}{{- end -}}
<ul class="nav nav-tabs" id="{{ $tab_set_id }}" role="tablist">
	{{- range $i, $e := $tabs -}}
	  {{- $id := printf "%s-%d" $tab_set_id $i -}}
	  {{- if (eq $i 0) -}}
		<li class="nav-item"><a data-toggle="tab" class="nav-link active" href="#{{ $id }}" role="tab" aria-controls="{{ $id }}" aria-selected="true">{{- trim .name " " -}}</a></li>
	  {{ else }}
		<li class="nav-item"><a data-toggle="tab" class="nav-link" href="#{{ $id }}" role="tab" aria-controls="{{ $id }}">{{- trim .name " " -}}</a></li>
	  {{- end -}}
{{- end -}}
</ul>
<div class="tab-content" id="{{ $tab_set_id }}">
{{- range $i, $e := $tabs -}}
{{- $id := printf "%s-%d" $tab_set_id $i -}}
{{- if (eq $i 0) -}}
  <div id="{{ $id }}" class="tab-pane show active" role="tabpanel" aria-labelledby="{{ $id }}">
{{ else }}
  <div id="{{ $id }}" class="tab-pane" role="tabpanel" aria-labelledby="{{ $id }}">
{{ end }}
<p>
	{{- with .content -}}
		{{- . -}}
	{{- else -}}
		{{- if eq $.Page.BundleType "leaf" -}}
			{{- /* find the file somewhere inside the bundle. Note the use of double asterisk */ -}}
			{{- with $.Page.Resources.GetMatch (printf "**%s*" .include) -}}
				{{- if ne .ResourceType "page" -}}
				{{- /* Assume it is a file that needs code highlighting. */ -}}
				{{- $codelang := $e.codelang | default ( path.Ext .Name | strings.TrimPrefix ".") -}}
				{{- highlight .Content $codelang "" -}}
				{{- else -}}
					{{- .Content -}}
				{{- end -}}
			{{- end -}}
		{{- else -}}
		{{- $path := path.Join $.Page.File.Dir .include -}}
		{{- $page := site.GetPage "page" $path -}}
		{{- with $page -}}
			{{- .Content -}}
		{{- else -}}
		{{- errorf "[%s] tabs include not found for path %q" site.Language.Lang $path -}}
		{{- end -}}
		{{- end -}}
	{{- end -}}
</div>
{{- end -}}
</div>
```

增加 `layouts/shortcodes/tabitem.html`

```
{{ if .Parent }}
	{{ $name := trim (.Get "name") " " }}
	{{ $include := trim (.Get "include") " "}}
	{{ $codelang := .Get "codelang" }}
	{{ if not (.Parent.Scratch.Get "tabs") }}
	{{ .Parent.Scratch.Set "tabs" slice }}
	{{ end }}
	{{ with .Inner }}
	{{ if $codelang }}
	{{ $.Parent.Scratch.Add "tabs" (dict "name" $name "content" (highlight . $codelang "") ) }}
	{{ else }}
	{{ $.Parent.Scratch.Add "tabs" (dict "name" $name "content" . ) }}
	{{ end }}
	{{ else }}
	{{ $.Parent.Scratch.Add "tabs" (dict "name" $name "include" $include "codelang" $codelang) }}
	{{ end }}
{{ else }}
	{{- errorf "[%s] %q: tab shortcode missing its parent" site.Language.Lang .Page.Path -}}
{{ end}}
```

用法：

```
{{< tabfolder name="tabfolder_name" >}}
{{% tabitem name="选项卡 1 名称" %}}
内容可以是 Markdown 也可以是 ``` 代码块
{{% /tabitem %}}
{{< tabitem name="选项卡 2 名称" codelang="程序语言名称" >}}
程序代码
{{< /tabitem >}}
{{< /tabfolder >}}
```

## Markdown 行内代码背景色

`assets/scss/_code.scss`

```scss
# 修改

p code, li > code, table code {
    color: inherit;
    padding: 0.2em 0.4em;
    margin: 0;
    font-size: 85%;
    word-break: normal;
    background-color: rgba($black, 0.1); // 原：background-color: rgba($black, 0.05);
    border-radius: $border-radius;

    br {
        display: none;
    }
}
```

## bootstrap 增加新的颜色

`assets/vendor/bootstrap/scss/_variables.scss`

```scss
$cosmic:    #0a1922 !default; // 新颜色
$blue:    #007bff !default;
$indigo:  #6610f2 !default;
$purple:  #6f42c1 !default;
$pink:    #e83e8c !default;
$red:     #dc3545 !default;
$orange:  #fd7e14 !default;
$yellow:  #ffc107 !default;
$green:   #28a745 !default;
$teal:    #20c997 !default;
$cyan:    #17a2b8 !default;

$colors: () !default;
$colors: map-merge(
  (
    "cosmic":     $cosmic, // 新颜色
    "blue":       $blue,
    "indigo":     $indigo,
    "purple":     $purple,
    "pink":       $pink,
    "red":        $red,
    "orange":     $orange,
    "yellow":     $yellow,
    "green":      $green,
    "teal":       $teal,
    "cyan":       $cyan,
    "white":      $white,
    "gray":       $gray-600,
    "gray-dark":  $gray-800
  ),
  $colors
);

$cosmos:    $cosmic !default; // 新颜色
$primary:       $blue !default;
$secondary:     $gray-600 !default;
$success:       $green !default;
$info:          $cyan !default;
$warning:       $yellow !default;
$danger:        $red !default;
$light:         $gray-100 !default;
$dark:          $gray-800 !default;

$theme-colors: () !default;
$theme-colors: map-merge(
  (
    "cosmos":    $cosmos,                 // 新颜色
    "primary":    $primary,
    "secondary":  $secondary,
    "success":    $success,
    "info":       $info,
    "warning":    $warning,
    "danger":     $danger,
    "light":      $light,
    "dark":       $dark
  ),
  $theme-colors
);
```

## [Partials] 支持 MathJax

新增 `layouts/partials/mathjax.html`

```html
{{ if .Params.math }}
<script>
  MathJax = {
    tex: {
      inlineMath: [["$", "$"]],
    },
    displayMath: [
      ["$$", "$$"],
      ["\[\[", "\]\]"],
    ],
    svg: {
      fontCache: "global",
    },
  };
</script>
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
{{ end }}
```

修改 `layouts/partials/head.html` 增加对 mathjax.html 的引入

```
{{ partial "mathjax.html" . }}
```

用法：在要使用公式的文档 front matter 中设置参数 `math: true` 开启公式脚本。`$...$` 为行内公式，`$$..$$` 为区块公式。


## [Partials] 显示备案号

修改 `layouts/partials/footer.hml` 增加——

```
{{ with .Site.Params.beian }}<p><small class="mt-1"><a href="https://beian.miit.gov.cn/" target="_blank">{{ .}}</a></small></p>{{ end }}
```

用法：config.toml 配置文件的 [params] 中设置 `beian = "备案号"` 参数。

## [Layouts] 布局：页面

新增布局 Page：`layouts/page/`

用法：front metter 中指定 `type = page`。

## 本地化 Google Fonts

`https://fonts.googleapis.com` 替换为 `https://fonts.loli.net`

文件：assets/scss/_variables.scss

```scss
// $web-font-path: "https://fonts.googleapis.com/css?family=#{$google_font_family}&display=swap";
$web-font-path: "https://fonts.loli.net/css?family=#{$google_font_family}&display=swap";
```

文件：assets/scss/rtl/_main.scss

```scss
    // @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@300;400;500;600;700&display=swap');
    @import url('https://fonts.loli.net/css2?family=Rubik:wght@300;400;500;600;700&display=swap');

    // @import url('https://fonts.googleapis.com/css2?family=Tajawal:wght@300;400;500;700&display=swap');
    @import url('https://fonts.loli.net/css2?family=Tajawal:wght@300;400;500;700&display=swap');
```