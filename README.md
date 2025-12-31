# PHPでのHTMLパース

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/) 

本ガイドでは、PHPでHTMLをパースする3つのテクニックを検証し、それぞれの強みと違いを比較します。

- [PHPでのHTMLパース](#parsing-html-with-php)
- [なぜPHPでHTMLをパースするのか？](#why-parse-html-in-php)
- [前提条件](#prerequisites)
- [PHPでのHTML取得](#html-retrieval-in-php)
- [PHPでのHTMLパース：3つのアプローチ](#html-parsing-in-php-3-approaches)
  - [アプローチ #1：`Dom\HTMLDocument`を使用](#approach-1-with-domhtmldocument)
  - [アプローチ #2：Simple HTML DOM Parserを使用](#approach-2-using-simple-html-dom-parser)
  - [アプローチ #3：SymfonyのDomCrawlerコンポーネントを使用](#approach-3-using-symfonys-domcrawler-component)
- [PHPでのHTMLパース：比較表](#parsing-html-in-php-comparison-table)
- [結論](#conclusion)

## なぜPHPでHTMLをパースするのか？

PHPでのHTMLパースとは、HTMLコンテンツをDOM（[Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)）構造に変換することです。DOM形式になれば、HTMLコンテンツのナビゲーションや操作を簡単に行えます。

特に、PHPでHTMLをパースする主な理由は次のとおりです。

- **データ抽出**：HTML要素のテキストや属性など、Webページから特定のコンテンツを取得します。  
- **自動化**：コンテンツのスクレイピング、レポート作成、HTMLからのデータ集約といったタスクを効率化します。  
- **サーバーサイドでのHTML処理**：アプリケーションでレンダリングする前に、HTMLをパースしてクリーンアップ、整形、変更します。  


## 前提条件

コーディングを始める前に、マシンに[PHP 8.4+](https://www.php.net/releases/8.4/en.php)がインストールされていることを確認してください。以下のコマンドを実行すると確認できます。

```bash
php -v
```

出力は次のようになります。

```
PHP 8.4.3 (cli) (built: Jan 19 2025 14:20:58) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.4.3, Copyright (c) Zend Technologies
    with Zend OPcache v8.4.3, Copyright (c), by Zend Technologies
```

次に、依存関係の管理を簡単にするためにComposerプロジェクトを初期化します。Composerがシステムにインストールされていない場合は、[ダウンロード](https://getcomposer.org/download/)してインストール手順に従ってください。

まず、PHPのHTMLプロジェクト用に新しいフォルダを作成します。

```bash
mkdir php-html-parser
```

ターミナルでそのフォルダに移動し、`composer init`コマンドで中にComposerプロジェクトを初期化します。

```bash
composer init
```

このプロセスではいくつか質問されます。デフォルトの回答で問題ありませんが、必要に応じて、PHPのHTMLパースプロジェクトに合わせてより具体的な内容を指定することもできます。

次に、お好みのIDEでプロジェクトフォルダを開きます。PHP開発には、[PHP拡張機能付きVisual Studio Code](https://code.visualstudio.com/docs/languages/php)または[IntelliJ WebStorm](https://www.jetbrains.com/webstorm/)が良い選択肢です。

続いて、プロジェクトフォルダに空の`index.php`ファイルを追加します。プロジェクト構成は次のようになります。

```
php-html-parser/
  ├── vendor/
  ├── composer.json
  └── index.php
```

`index.php`を開き、プロジェクトを初期化するために次のコードを追加してください。

```php
<?php

require_once __DIR__ . "/vendor/autoload.php";

// scraping logic...
```

次のコマンドでスクリプトを実行します。

```bash
php index.php
```

## PHPでのHTML取得

PHPでHTMLをパースする前に、パースするHTMLが必要です。本セクションでは、PHPでHTMLコンテンツにアクセスする2つの異なるアプローチを見ていきます。あわせて、[PHPでのWebスクレイピングガイド](https://brightdata.jp/blog/how-tos/web-scraping-php)も読むことをおすすめします。

### CURLを使用する

PHPは、HTTPリクエストを実行するために一般的に使われるHTTPクライアントであるcURLをネイティブでサポートしています。[cURL拡張を有効化](https://www.php.net/manual/en/book.curl.php)するか、Ubuntu Linuxでは次のコマンドでインストールしてください。

```bash
sudo apt-get install php8.4-curl
```

cURLを使用すると、オンラインサーバーへHTTP GETリクエストを送信し、サーバーから返されるHTMLドキュメントを取得できます。次の例のスクリプトは、シンプルなGETリクエストを行い、HTMLコンテンツを取得します。

```bash
// initialize cURL session
$ch = curl_init();

// set the URL you want to make a GET request to
curl_setopt($ch, CURLOPT_URL, "https://www.scrapethissite.com/pages/forms/?per_page=100");

// return the response instead of outputting it
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

// execute the cURL request and store the result in $response
$html = curl_exec($ch);

// close the cURL session
curl_close($ch);

// output the HTML response
echo $html;
```

上記のコードスニペットを`index.php`に追加して実行してください。次のHTMLコードが生成されます。

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hockey Teams: Forms, Searching and Pagination | Scrape This Site | A public sandbox for learning web scraping</title>
    <link rel="icon" type="image/png" href="/static/images/scraper-icon.png" />
    <!-- Omitted for brevity... -->
</html>
```

### ファイルから取得する

cURLを使って事前に取得した、[Scrape This Site](https://www.scrapethissite.com/pages/forms/?per_page=100)の「Hockey Teams」ページのHTMLを含む`index.html`というファイルがあると仮定します。

![The index.html file in the project folder](https://github.com/luminati-io/php-html-parsing/blob/main/Images/The-index.html-file-in-the-project-folder-2048x1216.png)

## PHPでのHTMLパース：3つのアプローチ

本セクションでは、PHPでHTMLをパースするために3つの異なるライブラリを使用する方法を説明します。

1. 素のPHP向けに`Dom\HTMLDocument`を使用
2. Simple HTML DOM Parserライブラリを使用
3. Symfonyの`DomCrawler`コンポーネントを使用

いずれのケースでも、ローカルの`index.html`ファイルからHTMLをパースし、ページ上のすべてのホッケーチームのエントリを選択して、そこからデータを抽出します。

![The table on the target page](https://github.com/luminati-io/php-html-parsing/blob/main/Images/The-table-on-the-target-page-2048x1107.png)

最終結果は、次の詳細を含むスクレイピング済みホッケーチームのエントリ一覧になります。

- Team Name
- Year
- Wins
- Losses
- Win %
- Goals For (GF)
- Goals Against (GA)
- Goal Difference

これらは、次の構造を持つHTMLテーブルから抽出できます。

![The HTML DOM structure of the table's rows](https://github.com/luminati-io/php-html-parsing/blob/main/Images/The-HTML-DOM-structure-of-the-tables-rows-2048x1134.png)

テーブル行の各カラムには固有のclassが付いており、classをCSSセレクタとして要素を選択し、テキストとして内容を取得することでデータを抽出できます。

## アプローチ #1：Dom\\HTMLDocumentを使用

PHP 8.4+ には組み込みの[`Dom\HTMLDocument`](https://www.php.net/manual/en/class.dom-htmldocument.php)クラスがあります。これはHTMLドキュメントを表し、HTMLコンテンツのパースやDOMツリーのナビゲーションを可能にします。

### ステップ #1：インストールとセットアップ

`Dom\HTMLDocument`は[Standard PHP Library](https://www.php.net/manual/en/book.spl.php)の一部です。ただし、使用するには[DOM拡張](https://www.php.net/manual/en/intro.dom.php)を有効化するか、次のLinuxコマンドでインストールする必要があります。

```bash
sudo apt-get install php-dom
```

### ステップ #2：HTMLパース

HTML文字列は次のようにパースできます。

```php
$dom = \DOM\HTMLDocument::createFromString($html);
```

`index.html`ファイルは次のようにパースできます。

```php
$dom = \DOM\HTMLDocument::createFromFile("./index.html");
```

`$dom`は[`Dom\HTMLDocument`](https://www.php.net/manual/en/class.dom-htmldocument.php)オブジェクトで、データパースに必要なメソッドが提供されます。

### ステップ #3：データパース

次のアプローチで、`\DOM\HTMLDocument`を使ってすべてのホッケーチームのエントリを選択できます。

```php
// select each row on the page
$table = $dom->getElementsByTagName("table")->item(0);
$rows = $table->getElementsByTagName("tr");

// iterate through each row and extract data
foreach ($rows as $row) {
  $cells = $row->getElementsByTagName("td");

  // extracting the data from each column
  $team = trim($cells->item(0)->textContent);
  $year = trim($cells->item(1)->textContent);
  $wins = trim($cells->item(2)->textContent);
  $losses = trim($cells->item(3)->textContent);
  $win_pct = trim($cells->item(5)->textContent);
  $goals_for = trim($cells->item(6)->textContent);
  $goals_against = trim($cells->item(7)->textContent);
  $goal_diff = trim($cells->item(8)->textContent);

  // create an array for the scraped team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print ("\n");
}
```

`\DOM\HTMLDocument`は高度なクエリメソッドを提供しません。そのため、`getElementsByTagName()`のようなメソッドと手動の反復処理に依存する必要があります。

使用したメソッドの内訳は次のとおりです。

- `getElementsByTagName()`：ドキュメント内の指定タグ（`<table>`、`<tr>`、`<td>`など）の全要素を取得します。
- `item()`：`getElementsByTagName()`が返す要素リストから個別要素を返します。
- `textContent`：要素の生のテキスト内容を返すプロパティで、表示されるデータ（チーム名、年など）を抽出できます。

また、テキスト内容の前後にある余分な空白を取り除いてデータを整えるために、[`trim()`](https://www.php.net/manual/en/function.trim.php)も使用しました。

上記スニペットを`index.php`に追加すると、次の結果が得られます。

```php
Array
(
    [team] => Boston Bruins
    [year] => 1990
    [wins] => 44
    [losses] => 24
    [win_pct] => 0.55
    [goals_for] => 299
    [goals_against] => 264
    [goal_diff] => 35
)

// omitted for brevity...

Array
(
    [team] => Detroit Red Wings
    [year] => 1994
    [wins] => 33
    [losses] => 11
    [win_pct] => 0.688
    [goals_for] => 180
    [goals_against] => 117
    [goal_diff] => 63
) 
```

## アプローチ #2：Simple HTML DOM Parserを使用

[Simple HTML DOM Parser](https://github.com/voku/simple_html_dom)は軽量なPHPライブラリで、HTMLコンテンツのパースと操作を簡単にします。

### ステップ #1：インストールとセットアップ

次のコマンドで、Composer経由でSimple HTML Dom Parserをインストールできます。

```php
composer require voku/simple_html_dom
```

または、`simple_html_dom.php`ファイルを手動でダウンロードしてプロジェクトに含めることもできます。

次に、このコード行で`index.php`にインポートします。

```php
use voku\helper\HtmlDomParser;
```

### ステップ #2：HTMLパース

HTML文字列をパースするには、`file_get_html()`メソッドを使用します。

```php
$dom = HtmlDomParser::str_get_html($html);
```

`index.html`をパースする場合は、代わりに`file_get_html()`を書きます。

```php
$dom = HtmlDomParser::file_get_html($str);
```

これによりHTMLコンテンツが`$dom`オブジェクトに読み込まれ、DOMを簡単に辿れるようになります。

### ステップ #3：データパース

Simple HTML DOM Parserを使用して、HTMLからホッケーチームのデータを抽出します。

```php
// find all rows in the table
$rows = $dom->findMulti("table tr.team");

// loop through each row to extract the data
foreach ($rows as $row) {
  // extract data using CSS selectors
  $team_element = $row->findOne(".name");
  $team = trim($team_element->plaintext);

  $year_element = $row->findOne(".year");
  $year = trim($year_element->plaintext);

  $wins_element = $row->findOne(".wins");
  $wins = trim($wins_element->plaintext);

  $losses_element = $row->findOne(".losses");
  $losses = trim($losses_element->plaintext);

  $win_pct_element = $row->findOne(".pct");
  $win_pct = trim($win_pct_element->plaintext);

  $goals_for_element = $row->findOne(".gf");
  $goals_for = trim($goals_for_element->plaintext);

  $goals_against_element = $row->findOne(".ga");
  $goals_against = trim(string: $goals_against_element->plaintext);

  $goal_diff_element = $row->findOne(".diff");
  $goal_diff = trim(string: $goal_diff_element->plaintext);

  // create an array with the extracted team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print("\n");
}
```

上記で使用したSimple HTML DOM Parserの機能は次のとおりです。

- `findMulti()`：指定したCSSセレクタで識別されるすべての要素を選択します。
- `findOne()`：指定したCSSセレクタに一致する最初の要素を特定します。
- `plaintext`：HTML要素内部の生のテキスト内容を取得するための属性です。

今回は、より包括的で堅牢なロジックでCSSセレクタを適用しましたが、結果は最初のPHPでのHTMLパースアプローチと同じです。

## アプローチ #3：SymfonyのDomCrawlerコンポーネントを使用

[Symfonyの`DomCrawler`コンポーネント](https://symfony.com/doc/current/components/dom_crawler.html)は、HTMLドキュメントを簡単にパースし、そこからデータを抽出できるようにします。

> **Note**:
> このコンポーネントは[Symfony framework](https://symfony.com/)の一部ですが、本セクションで行うようにスタンドアロンでも使用できます。

### ステップ #1：インストールとセットアップ

次のComposerコマンドでSymfonyの`DomCrawler`コンポーネントをインストールします。

```bash
composer require symfony/dom-crawler
```

次に、`index.php`ファイルにインポートします。

```php
use Symfony\Component\DomCrawler\Crawler;
```

### ステップ #2：HTMLパース

HTML文字列をパースするには、`html()`メソッドで[`Crawler`](https://github.com/symfony/symfony/blob/7.2/src/Symfony/Component/DomCrawler/Crawler.php)インスタンスを作成します。

```php
$crawler = new Crawler($html);
```

ファイルをパースする場合は、`file_get_contents()`を使用して`Crawler`インスタンスを作成します。

```php
$crawler = new Crawler(file_get_contents("./index.html"));
```

上記の行によりHTMLコンテンツが`$crawler`オブジェクトに読み込まれ、データの走査と抽出を容易にするメソッドが提供されます。

### ステップ #3：データパース

`DomCrawler`コンポーネントを使用してホッケーチームのデータを抽出します。

```php
// select all rows within the table
$rows = $crawler->filter("table tr.team");

// loop through each row to extract the data
$rows->each(function ($row, $i) {
  // extract data using CSS selectors
  $team_element = $row->filter(".name");
  $team = trim($team_element->text());

  $year_element = $row->filter(".year");
  $year = trim($year_element->text());

  $wins_element = $row->filter(".wins");
  $wins = trim($wins_element->text());

  $losses_element = $row->filter(".losses");
  $losses = trim($losses_element->text());

  $win_pct_element = $row->filter(".pct");
  $win_pct = trim($win_pct_element->text());

  $goals_for_element = $row->filter(".gf");
  $goals_for = trim($goals_for_element->text());

  $goals_against_element = $row->filter(".ga");
  $goals_against = trim($goals_against_element->text());

  $goal_diff_element = $row->filter(".diff");
  $goal_diff = trim($goal_diff_element->text());

  // create an array with the extracted team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print ("\n");
});
```

使用した`DomCrawler`のメソッドは次のとおりです。

- `each()`：選択した要素のリストを反復処理します。
- `filter()`：CSSセレクタに基づいて要素を選択します。
- `text()`：選択した要素のテキスト内容を抽出します。

## PHPでのHTMLパース：比較表

ここで取り上げたPHPでHTMLをパースする3つのアプローチを、以下のサマリ表で比較できます。

|     | **\\DOM\\HTMLDocument** | **Simple HTML DOM Parser** | **Symfony’s DomCrawler** |
| --- | --- | --- | --- |
| **Type** | PHPネイティブコンポーネント | 外部ライブラリ | Symfonyコンポーネント |
| **GitHub Stars** | —   | 880+ | 4,000+ |
| **XPath Support** | ❌   | ✔️  | ✔️  |
| **CSS Selector Support** | ❌   | ✔️  | ✔️  |
| **Learning Curve** | 低 | 低〜中 | 中 |
| **Simplicity of Use** | 中 | 高 | 高 |
| **API** | 基本 | 充実 | 充実 |

## 結論

これらのソリューションは動作しますが、ターゲットのWebページがレンダリングにJavaScriptを依存している場合は効果的ではありません。そのようなケースでは、上記のようなシンプルなHTMLパース手法だけでは不十分です。代わりに、[Scraping Browser](https://brightdata.jp/products/scraping-browser)のような、高度なHTMLパース機能を備えたフル機能のスクレイピングブラウザが必要になります。

HTMLパースを省略して構造化データへ即時アクセスしたい場合は、数百のWebサイトを網羅する[すぐに使えるデータセット](https://brightdata.jp/products/datasets)をご覧ください。

今すぐBright Dataアカウントを作成し、無料トライアルで当社のデータおよびスクレイピングソリューションのテストを開始しましょう！