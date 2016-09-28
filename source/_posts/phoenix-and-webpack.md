---
title: Phoenix and Webpack
date: 2016-09-24 09:35:36
tags:
---

# 동기

- 브런치보다 웹팩이 익숙하고 유지보수 잘되고 유저가 많다.
- 이왕할거면 초기에 하는게 나중에 하는거보다 덜 골치아프다.

# 환경

사용한 버전은 이렇다.

- Elixir 1.3.3
- Phoenix 1.2.1
- Node 6.6.0

# 과정

브런치 없이 설치하는게 더 귀찮은데, 이미 오래전에 없이 설치해버려서.. 그냥 둘 다
설명하겠다.

[소스 코드](https://github.com/kenips/phoenix/blob/master/installer/lib/phoenix_new.ex#L48-L57)를
[확인해도](https://github.com/phoenixframework/phoenix/blob/master/installer/lib/phoenix_new.ex#L70-L76)
알 수 있지만 .gitignore의 내용이 다른걸 재외하면, 두 방식의 파일 차이는 이 정도
이다.

```diff
-* creating with_brunch/brunch-config.js
-* creating with_brunch/package.json
-* creating with_brunch/web/static/css/app.css
-* creating with_brunch/web/static/css/phoenix.css
-* creating with_brunch/web/static/assets/favicon.ico
-* creating with_brunch/web/static/assets/images/phoenix.png
-* creating with_brunch/web/static/js/app.js
-* creating with_brunch/web/static/js/socket.js
-* creating with_brunch/web/static/assets/robots.txt
+* creating without_brunch/priv/static/css/app.css
+* creating without_brunch/priv/static/favicon.ico
+* creating without_brunch/priv/static/images/phoenix.png
+* creating without_brunch/priv/static/js/app.js
+* creating without_brunch/priv/static/js/phoenix.js
+* creating without_brunch/priv/static/robots.txt
```

레일스 감각으로 말하면, web/static은 app/assets이고 priv/static은 public/assets
다. frontend같은 다른 경로를 사용해도 되긴하는데 스포클렛처럼 타이트하게 묶여
있거나 묵시적으로 처리하는 부분이 있는게 아니라, 두 경우 모두 web/static을
사용하도록 하겠다.

## 파일 구조 잡기

### 브런치 없이 설치한 경우

```bash
$ mix phoenix.new without_brunch --no-brunch
* creating without_brunch/config/config.exs
* creating without_brunch/config/dev.exs
* creating without_brunch/config/prod.exs
* creating without_brunch/config/prod.secret.exs
* creating without_brunch/config/test.exs
* creating without_brunch/lib/without_brunch.ex
* creating without_brunch/lib/without_brunch/endpoint.ex
* creating without_brunch/test/views/error_view_test.exs
* creating without_brunch/test/support/conn_case.ex
* creating without_brunch/test/support/channel_case.ex
* creating without_brunch/test/test_helper.exs
* creating without_brunch/web/channels/user_socket.ex
* creating without_brunch/web/router.ex
* creating without_brunch/web/views/error_view.ex
* creating without_brunch/web/web.ex
* creating without_brunch/mix.exs
* creating without_brunch/README.md
* creating without_brunch/web/gettext.ex
* creating without_brunch/priv/gettext/errors.pot
* creating without_brunch/priv/gettext/en/LC_MESSAGES/errors.po
* creating without_brunch/web/views/error_helpers.ex
* creating without_brunch/lib/without_brunch/repo.ex
* creating without_brunch/test/support/model_case.ex
* creating without_brunch/priv/repo/seeds.exs
* creating without_brunch/.gitignore
* creating without_brunch/priv/static/css/app.css
* creating without_brunch/priv/static/js/app.js
* creating without_brunch/priv/static/robots.txt
* creating without_brunch/priv/static/js/phoenix.js
* creating without_brunch/priv/static/images/phoenix.png
* creating without_brunch/priv/static/favicon.ico
* creating without_brunch/test/controllers/page_controller_test.exs
* creating without_brunch/test/views/layout_view_test.exs
* creating without_brunch/test/views/page_view_test.exs
* creating without_brunch/web/controllers/page_controller.ex
* creating without_brunch/web/templates/layout/app.html.eex
* creating without_brunch/web/templates/page/index.html.eex
* creating without_brunch/web/views/layout_view.ex
* creating without_brunch/web/views/page_view.ex
```

먼저 .gitignore에 node_modules, priv/static/ 을 추가한다.
이 위치는 컴파일 후의 스테틱 파일이 올자리이다.

```bash
# Static artifacts
/node_modules

# Since we are building assets from web/static,
# we ignore priv/static. You may want to comment
# this depending on your deployment strategy.
/priv/static/
```

다음은 파일을 옮기고 필요없는 파일을 지운다.

```bash
mkdir -p web/static/assets/images web/static/js web/static/css
mv priv/static/images/phoenix.png web/static/assets/images/phoenix.png
mv priv/static/robots.txt web/static/assets/robots.txt
mv priv/static/js/app.js web/static/js/app.js
mv priv/static/favicon.ico web/static/assets/favicon.ico
touch web/static/css/app.css
rm -rf priv/static/*
```

### 그냥 설치한 경우

```bash
$ mix phoenix.new with_brunch
* creating with_brunch/config/config.exs
* creating with_brunch/config/dev.exs
* creating with_brunch/config/prod.exs
* creating with_brunch/config/prod.secret.exs
* creating with_brunch/config/test.exs
* creating with_brunch/lib/with_brunch.ex
* creating with_brunch/lib/with_brunch/endpoint.ex
* creating with_brunch/test/views/error_view_test.exs
* creating with_brunch/test/support/conn_case.ex
* creating with_brunch/test/support/channel_case.ex
* creating with_brunch/test/test_helper.exs
* creating with_brunch/web/channels/user_socket.ex
* creating with_brunch/web/router.ex
* creating with_brunch/web/views/error_view.ex
* creating with_brunch/web/web.ex
* creating with_brunch/mix.exs
* creating with_brunch/README.md
* creating with_brunch/web/gettext.ex
* creating with_brunch/priv/gettext/errors.pot
* creating with_brunch/priv/gettext/en/LC_MESSAGES/errors.po
* creating with_brunch/web/views/error_helpers.ex
* creating with_brunch/lib/with_brunch/repo.ex
* creating with_brunch/test/support/model_case.ex
* creating with_brunch/priv/repo/seeds.exs
* creating with_brunch/.gitignore
* creating with_brunch/brunch-config.js
* creating with_brunch/package.json
* creating with_brunch/web/static/css/app.css
* creating with_brunch/web/static/css/phoenix.css
* creating with_brunch/web/static/js/app.js
* creating with_brunch/web/static/js/socket.js
* creating with_brunch/web/static/assets/robots.txt
* creating with_brunch/web/static/assets/images/phoenix.png
* creating with_brunch/web/static/assets/favicon.ico
* creating with_brunch/test/controllers/page_controller_test.exs
* creating with_brunch/test/views/layout_view_test.exs
* creating with_brunch/test/views/page_view_test.exs
* creating with_brunch/web/controllers/page_controller.ex
* creating with_brunch/web/templates/layout/app.html.eex
* creating with_brunch/web/templates/page/index.html.eex
* creating with_brunch/web/views/layout_view.ex
* creating with_brunch/web/views/page_view.ex
```

일단 필요없는 파일을 지운다.

```bash
rm brunch-config.js package.json
rm web/static/css/phoenix.css web/static/js/socket.js
```

### npm, webpack 설정

다음 명령을 실행하고 적당히 내용 입력하면 package.json 파일이 만들어진다.

```bash
npm init
```

webpack을 설치하고 설정 파일을 만든다.

```bash
npm install --save-dev webpack babel-preset-es2015 copy-webpack-plugin \
                       babel-loader babel-core \
                       css-loader extract-text-webpack-plugin style-loader
```

webpack.config.js파일은 이런 내용이 들어가면 된다.

```js
const ExtractTextPlugin = require("extract-text-webpack-plugin")
const CopyWebpackPlugin = require("copy-webpack-plugin")
module.exports = {
  entry: ["./web/static/css/app.css",
          "./web/static/js/app.js"],
  output: {
    path: "./priv/static",
    filename: "js/app.js"
  },
  module: {
    loaders: [{
      test: /\.js$/,
      exclude: /node_modules/,
      include: __dirname,
      loader: ["babel"],
      query: {
        presets: ["es2015"]
      }
    }, {
      test: /\.css$/,
      loader: ExtractTextPlugin.extract("style", "css")
    }]
  },
  plugins: [
    new ExtractTextPlugin("css/app.css"),
    new CopyWebpackPlugin([{ from: "./web/static/assets" }])
  ],
  resolve: {
    modulesDirectories: [ "node_modules", __dirname + "/web/static/js" ]
  }
}
```

일단 정적파일의 복사, js, css의 생성까지만 다루는 단순한 설정이다.

package.json에 실행 옵션을 포함한 단축명령을 적는다.

```json
  "scripts": {
    "start": "webpack --watch-stdin --progress --color",
    "compile": "webpack -p"
  },
```

방금 생성한 명령을 watcher에 추가해 파일 수정이 있을때마다 실행 시킬 수 있다.

config/dev.exs에 넣어두자

```elixir
config :app_name, AppName.Endpoint,
  # 다른 설정은 그대로 둘것
  watchers: [npm: ["start"]]
```

## phoenix 자바스크립트들

npm에 등록되어있으니 그냥 설치하면된다.

```bash
npm install file:deps/phoenix_html file:deps/phoenix --save
```

위에 패스도 잡아뒀으니, 이전처럼 import해서 사용할 수 있다.

# ref

<http://matthewlehner.net/using-webpack-with-phoenix-and-elixir/>
<http://mikker.github.io/2016/02/04/updated-phoenix-webpack-react-setup.html>
