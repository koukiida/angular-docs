<!-- TOC -->

- [サービスのテスト](#サービスのテスト)
    - [Angular のテスティングユーティリティ](#angular-のテスティングユーティリティ)
    - [describe](#describe)
        - [テストスイート](#テストスイート)
    - [it](#it)
        - [テストスペック](#テストスペック)
    - [beforeEach](#beforeeach)
    - [expect(hoge).toBe(fuga)](#expecthogetobefuga)
        - [アサーション（Assertion）](#アサーションassertion)
        - [Matcher](#matcher)
    - [DoneFn](#donefn)
    - [#getValue should return faked value from a fakeService](#getvalue-should-return-faked-value-from-a-fakeservice)
    - [#getValue should return faked value from a fake object](#getvalue-should-return-faked-value-from-a-fake-object)
    - [#getValue should return stubbed value from a spy](#getvalue-should-return-stubbed-value-from-a-spy)
        - [テストダブル](#テストダブル)
        - [Spy（スパイ）](#spyスパイ)
    - [TestBed](#testbed)
        - [TestBed.configureTestingModule()](#testbedconfiguretestingmodule)
    - [他のテストの流派](#他のテストの流派)
    - [setup()](#setup)
    - [分割代入](#分割代入)
    - [HttpClient の注入](#httpclient-の注入)
    - [Observable をサブスクライブする必要があります。](#observable-をサブスクライブする必要があります)
    - [HttpClientTestingModule](#httpclienttestingmodule)

<!-- /TOC -->

<a id="markdown-サービスのテスト" name="サービスのテスト"></a>
# サービスのテスト

<a id="markdown-angular-のテスティングユーティリティ" name="angular-のテスティングユーティリティ"></a>
## Angular のテスティングユーティリティ

`import { TestBed } from '@angular/core/testing';`

こういう奴かな, Angular で用意されている Angular でテストコードを書くときに便利なモジュールのこと？

<a id="markdown-describe" name="describe"></a>
## describe

Jasmine における`テストスイート`

describe にはスイートのタイトルと実行するコードブロック(関数)を引き渡す

<a id="markdown-テストスイート" name="テストスイート"></a>
### テストスイート

いくつかのテストをまとめたもの

= テストクラス

<a id="markdown-it" name="it"></a>
## it

Jasmine における`テストスペック`

it にはスペックのタイトルと実行するコードブロックを引き渡す

<a id="markdown-テストスペック" name="テストスペック"></a>
### テストスペック

テストの本体

= テストメソッド、テストケース

<a id="markdown-beforeeach" name="beforeeach"></a>
## beforeEach

そのスイートの各テストの実行前に処理したい内容を記述したコードブロックを引き渡す
> ここで記述した処理はスイートに存在するすべてのスペックの開始前で実行される

<a id="markdown-expecthogetobefuga" name="expecthogetobefuga"></a>
## expect(hoge).toBe(fuga)

Jasmine における `アサーション` の記述

hoge が評価対象の値、fuga が期待値

toBe は `Matcher` のメソッドの一つで厳密等価(===)であるかの比較を行う

<a id="markdown-アサーションassertion" name="アサーションassertion"></a>
### アサーション（Assertion）

実行結果（実績値）と期待値を用いて評価を行うための文

= エクセプション（Exception）

<a id="markdown-matcher" name="matcher"></a>
### Matcher

テストの評価条件を定義するもの

Jasmine の組み込みの Matcher には toBe 以外にも様々なメソッドがある
> toBeLess(Greater)Than: 大小比較, toThrow: ターゲットの関数内で例外が Throw されたかどうか...

<a id="markdown-donefn" name="donefn"></a>
## DoneFn

(?_?)

<a id="markdown-getvalue-should-return-faked-value-from-a-fakeservice" name="getvalue-should-return-faked-value-from-a-fakeservice"></a>
## #getValue should return faked value from a fakeService

```ts
it('#getValue should return faked value from a fakeService', () => {
    masterService = new MasterService(new FakeValueService());
    expect(masterService.getValue()).toBe('faked service value');
});
```

本物の ValueService と同じ機能をもつような（その実装は同じでなくてもいい）ダミーの（fake）サービス（class）を用いて依存性の解決を行うパターン

fake を使う理由
> MasterService クラスの getValue メソッドの振る舞いを検証（ユニットテスト）するのに MasterService の依存する ValueService の振る舞い（実装）が影響してはいけないということ...だと思う

<a id="markdown-getvalue-should-return-faked-value-from-a-fake-object" name="getvalue-should-return-faked-value-from-a-fake-object"></a>
## #getValue should return faked value from a fake object

```ts
it('#getValue should return faked value from a fake object', () => {
    const fake =  { getValue: () => 'fake value' };
    masterService = new MasterService(fake as ValueService);
    expect(masterService.getValue()).toBe('fake value');
});
```

fake をオブジェクトで表現しているだけで FakeValueService を使うパターンと同じ

<a id="markdown-getvalue-should-return-stubbed-value-from-a-spy" name="getvalue-should-return-stubbed-value-from-a-spy"></a>
## #getValue should return stubbed value from a spy

```ts
it('#getValue should return stubbed value from a spy', () => {
    // create `getValue` spy on an object representing the ValueService
    const valueServiceSpy =
      jasmine.createSpyObj('ValueService', ['getValue']);
 
    // set the value to return when the `getValue` spy is called.
    const stubValue = 'stub value';
    valueServiceSpy.getValue.and.returnValue(stubValue);
 
    masterService = new MasterService(valueServiceSpy);
 
    expect(masterService.getValue())
      .toBe(stubValue, 'service returned stub value');
    expect(valueServiceSpy.getValue.calls.count())
      .toBe(1, 'spy method was called once');
    expect(valueServiceSpy.getValue.calls.mostRecent().returnValue)
      .toBe(stubValue);
  });
});
```

上の2つのパターンで使用した本物のサービス（とそのメソッド）を fake クラス（オブジェクト）のような仕組み（そのもの）は`テストダブル`と呼ばれることがあります

このパターンではそのテストダブルを Jasmine から提供される機能を使用して検証を行っています。このパターンの場合、Jasmine の機能によって（返却）値の検査だけでなくメソッドの実行回数や引数などさらに詳細なテストをすることができます

<a id="markdown-テストダブル" name="テストダブル"></a>
### テストダブル

テストダブル (Test Double) とは、ソフトウェアテストにおいて、テスト対象が依存しているコンポーネントを置き換える代用品のこと by Wikipedia

+ スタブ
    > 値っぽい
+ スパイ
    > メソッドっぽい
+ モック
+ フェイク
+ ダミー
    > 値っぽい

というようにその性質によっていくつかのパターンがある…が、これらの用語は巷にある資料などで必ずしも正しく使用されているとは限らず、そもそもの定義が場所や文脈で曖昧なので注意が必要。これらについてはあまり用語と定義にこだわりすぎずその場で意図されている内容を読み取ることができる程度に自分なりに理解しておくにとどめるほうがよさそう

この資料では Jasmine から提供されている機能という意味でこれらの用語を用いる

<a id="markdown-spyスパイ" name="spyスパイ"></a>
### Spy（スパイ）

Jasmine では、あるメソッドに対するテストダブルを定義するための Spy クラスが提供されている

返り値の設定もできるためスタブ的な役割も持てる

<a id="markdown-testbed" name="testbed"></a>
## TestBed

[Angular のテスティングユーティリティ](#angular-のテスティングユーティリティ)のクラスの一つ

`@NgModule` をエミュレートするということなのでテストを実行するためのモジュール（環境）を作成してくれる

<a id="markdown-testbedconfiguretestingmodule" name="testbedconfiguretestingmodule"></a>
### TestBed.configureTestingModule()

```ts
// TestBed.configureTestingModule()メソッドは@NgModuleで渡すことができるプロパティとほぼ同じプロパティをもつメタデータオブジェクトを受け取ります。
// ってことは

TestBed.configureTestingModule({ ~ここは同じ~ })

@NgModule({ ~ここは同じ~ })

// ってこと?
```
TestBed を使用しない場合と比べると

```ts
const valueServiceSpy =
      jasmine.createSpyObj('ValueService', ['getValue']);
...
masterService = new MasterService(valueServiceSpy);
...
```
と
```ts
...
  const spy = jasmine.createSpyObj('ValueService', ['getValue']);

  TestBed.configureTestingModule({
    // Provide both the service-to-test and its (spy) dependency
    providers: [
      MasterService,
      { provide: ValueService, useValue: spy }
    ]
  });
  // Inject both the service-to-test and its (spy) dependency
  masterService = TestBed.get(MasterService);
...
```
が一緒？

<a id="markdown-他のテストの流派" name="他のテストの流派"></a>
## 他のテストの流派

「beforeEach()を使用せずにテストする」他のテストの流派

?

<a id="markdown-setup" name="setup"></a>
## setup()

他の流派ではスイート内に `beforeEach()` の代わりにこのような `setup` 関数を用意する

スイート内で共通な準グローバルといえるような変数を定義する必要がなくなる

他の流派ではさらに `TestBed` をしないパターンを好むらしい

```ts
function setup() {
  const valueServiceSpy =
    jasmine.createSpyObj('ValueService', ['getValue']);
  const stubValue = 'stub value';
  const masterService = new MasterService(valueServiceSpy);

  valueServiceSpy.getValue.and.returnValue(stubValue);
  return { masterService, stubValue, valueServiceSpy };
}
```

<a id="markdown-分割代入" name="分割代入"></a>
## 分割代入

= アンパック代入?

<a id="markdown-httpclient-の注入" name="httpclient-の注入"></a>
## HttpClient の注入

ここではこれまでと同様に HttpClient のスパイを使用するパターンを説明している（Angular のテスティングユーティリティを使っていない）

`http method` の種類、呼び出し回数、取得データの検査が行われる

HeroService.getHeroes() の実装
```ts
@Injectable()
export class HeroService {

    readonly heroesUrl = 'api/heroes';  // URL to web api

    constructor(private http: HttpClient) { }

    /** GET heroes from the server */
    getHeroes (): Observable<Hero[]> {
        return this.http.get<Hero[]>(this.heroesUrl)
        .pipe(
            tap(heroes => this.log(`fetched heroes`)),
            catchError(this.handleError('getHeroes'))
        ) as Observable<Hero[]>;
    }
    ...
}
```

<a id="markdown-observable-をサブスクライブする必要があります" name="observable-をサブスクライブする必要があります"></a>
## Observable をサブスクライブする必要があります。

Observable.subscribe() 内でアサーションを記述する場合、成功(next)/失敗(error)のコールバックをどちらも記述することが必要。
> 例えば get の成功パターンの検査（一つ目のスペック）であっても subscribe() に対して失敗時のコールバックを記述する必要がある。逆もしかり。

<a id="markdown-httpclienttestingmodule" name="httpclienttestingmodule"></a>
## HttpClientTestingModule

テスティングユーティリティを使用するパターンについては[Httpガイド(angular.jpに飛びます)](https://angular.jp/guide/http#testing-http-requests)で説明されている。