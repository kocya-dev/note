# Playwright

Webテストフレームワーク。  
Chromium、WebKit、Firefoxなどのエンジンをサポート。  
SeleniumなどのWebDriverを用いた仕組みではない。  
非同期、イベント駆動型アーキテクチャで、タイムアウトの仕組みに依存せず、UIの変化を自動的に待ってテスト処理を進めることができる。

[公式](https://playwright.dev/)  
[API](https://playwright.dev/docs/api/class-playwright)

---
## 環境構築、プロジェクト作成
```
npm init playright@latest
```
以下のように対話形式でプロジェクトの情報を入力する。
```
Need to install the following packages:
  create-playwright@1.17.125
Ok to proceed? (y) y
Getting started with writing end-to-end tests with Playwright:
Initializing project in '.'
√ Do you want to use TypeScript or JavaScript? · TypeScript
√ Where to put your end-to-end tests? · tests
√ Add a GitHub Actions workflow? (y/N) · false
√ Install Playwright browsers (can be done manually via 'npx playwright install')? (Y/n) · true
Installing Playwright Test (npm install --save-dev @playwright/test)…
```
---
## プロジェクト構成
```
root
  ├── node_modules
  ├── playwright-report
  │  └── index.html
  ├── tests
  │  └── example.spec.ts        // テストコード
  ├── test-examples
  │  └── demo-todo-app.spec.ts  // サンプルテストコード
  ├── .gitignore
  ├── package-lock.json         // npmでインストールしたパッケージの詳細情報
  ├── package.json              // プロジェクト設定。依存するパッケージおよびバージョン情報
  └── playwright.config.ts      // テスト実行に関わる設定
```
---
## 操作
```
  npx playwright test
    Runs the end-to-end tests.

  npx playwright test --project=chromium
    Runs the tests only on Desktop Chrome.

  npx playwright test example
    Runs the tests in a specific file.

  npx playwright test --debug
    Runs the tests in debug mode.

  npx playwright codegen
    Auto generate tests with Codegen.
```

codegenでブラウザを起動して基本操作を記録しつつ、    
そこで生成したコードに対する検証コードを手動で記載するのが大きな流れ。  

ヘッドレスモードを無効にするには下記コマンド。
```
npx playwright test --headed
```

## コード
* locaterで位置を特定する
* assertionで検証する

---
## CI/CD

CI GitHub Actions  
https://playwright.dev/docs/ci-intro

---
## action
### Text Input
https://playwright.dev/docs/input#text-input
```js
// Text input
await page.getByRole('textbox').fill('Peter');

// Date input
await page.getByLabel('Birth date').fill('2020-02-02');

// Time input
await page.getByLabel('Appointment time').fill('13:15');

// Local datetime input
await page.getByLabel('Local time').fill('2020-03-02T05:15')
```
### Mouse
https://playwright.dev/docs/input#mouse-click
```js
// Generic click
await page.getByRole('button').click();

// Double click
await page.getByText('Item').dblclick();

// Right click
await page.getByText('Item').click({ button: 'right' });

// Shift + click
await page.getByText('Item').click({ modifiers: ['Shift'] });

// Hover over element
await page.getByText('Item').hover();

// Click the top left corner
await page.getByText('Item').click({ position: { x: 0, y: 0} });
```

### Key  
https://playwright.dev/docs/input#keys-and-shortcuts

```js
locator.type('入力文字列')

// Type character by character
await page.locator('#area').type('Hello World!');

locator.press('キー名')

// example
await page.getByText('Submit').press('Enter');
await page.locator('#name').press('Shift+A');
```

### Checkbox, Radio button
https://playwright.dev/docs/input#checkboxes-and-radio-buttons

### Select option
https://playwright.dev/docs/input#select-options

### Focus
https://playwright.dev/docs/input#focus-element
```js
await page.getByLabel('Password').focus();
```

---
## 参考

Doc  
https://playwright.dev/docs/intro  
API  
https://playwright.dev/docs/api/class-playwright  

https://minerva.mamansoft.net/%F0%9F%93%98Articles/%F0%9F%93%983%E5%B9%B4%E7%9B%AE%E3%80%813%E5%9B%9E%E7%9B%AE%E3%81%AEPlaywright%E8%A8%98%E4%BA%8B%E3%82%92%E6%9B%B8%E3%81%8F
