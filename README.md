# Laravel-api-with-chatgpt

## 目的

ChatGPTの応答を頼りにどこまで目的のアプリケーションが実装できるか、実際にChatGPTを駆使しながら行ってみる。

コードは基本的にはChatGPTからの出力のコピー＆ペーストで、ChatGPTの枠組みの外で調査をしたり実装方法を検索することは極力行わない。

状況に応じて多少のアレンジは許容する。

## 機能要件

### Laravelを使ったRESTfulAPIを作る。

私自身がほとんどLaravelを触ってことがないから（PHPは習得している）。未知の言語、フレームワークでのプログラミングをChatGPTを使って行うという体で行う。

### OAuth2による認可を実装し、認可したアカウントに対してCRUDが可能な何らかの機能を実装する。

- アカウントの新規作成。ユーザープロファイルが作成される。
- ログイン/ログアウト
- ユーザープロファイルの読み込み（自アカウントしかできない）
- ユーザープロファイルの変更（何らかのバリデーションも実装）
- ユーザーの削除（アカウントごと削除する）

## 実装実況

### step1 Laravelのインストール
指示をコピペするのみ。
```
composer create-project --prefer-dist laravel/laravel app_name
```

### step2 データベースの作成
ChatGPTではMySQLに接続する設定ファイルを出力しているが、今回はSQLiteを使う。
デフォルトがMySQLだったので、SQLiteにする設定をChatGPTに聞いて.envを修正。

```
DB_CONNECTION=sqlite
DB_DATABASE=./database.sqlite
# 以下DB設定はコメントアウト
```

### step3 マイグレーションファイルの作成

```
php artisan make:migration create_users_table
```
NP.

### ステップ4: マイグレーションの実行

```
php artisan migrate
```

エラーが発生。

```
  Creating migration table .............................................................................................................. 6ms DONE

   INFO  Running migrations.  

  2014_10_12_000000_create_users_table .................................................................................................. 3ms DONE
  2014_10_12_100000_create_password_reset_tokens_table .................................................................................. 1ms DONE
  2019_08_19_000000_create_failed_jobs_table ............................................................................................ 2ms DONE
  2019_12_14_000001_create_personal_access_tokens_table ................................................................................. 2ms DONE
  2023_05_07_123527_create_users_table .................................................................................................. 0ms FAIL

   Illuminate\Database\QueryException 

  SQLSTATE[HY000]: General error: 1 table "users" already exists (Connection: sqlite, SQL: create table "users" ("id" integer primary key autoincrement not null, "created_at" datetime, "updated_at" datetime))
  ```

  `users`というテーブルはすでに存在する（laravelインストール時の migrationにすでに入っている）にもかかわらず改めて`users`テーブルを作成しようとしてエラーになっている。

  プロンプト＞
  ```
  ステップ4にて、SQLSTATE[HY000]: General error: 1 table "users" already exists (Connection: sqlite, SQL: create table "users" ("id" integer primary key autoincrement not null, "created_at" datetime, "updated_at" datetime)) というエラーが発生しました。どのように対処すれば良いですか。
  ```

  回答は、マイグレーションをリセットして再度migrateしろとのこと。この場合ステップ3で新しくusersテーブルを作成したのが原因なので、resetしたのちステップ3の実施内容を無かったことにしてmigrateするしかない。

```
  php artisan migrate:reset
```
この後、ステップ3で作成したマイグレーションファイルを削除する。

この時点でもうChatGPTの範囲外で対策を調べている。

一旦この時点で正常に稼働するかチェックする。

```
php artisan serve
```

http://localhost:8000/ にてスタートページが表示された。稼働に問題はなさそう。

### ステップ5: ルートの定義

アプリケーションのエンドポイント作成

```
Route::get('/', function () {
    return view('welcome');
});

Route::get('/users', 'UserController@index');
Route::get('/users/{id}', 'UserController@show');
Route::post('/users', 'UserController@store');
Route::put('/users/{id}', 'UserController@update');
Route::delete('/users/{id}', 'UserController@destroy');
```

`routes/api.php`に追加。

### ステップ6: コントローラの作成

```
php artisan make:controller UserController
```

