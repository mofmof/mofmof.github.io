---
layout: blog
title: Rails標準のGeneratorを拡張して自作Generatorを作る
category: blog
tags: [rails,generator]  
summary: scaffoldで生成されるものをちょっとカスタマイズしたいなーってときあるじゃないですか。
author: aharada
---

scaffoldで生成されるものをちょっとカスタマイズしたいなーってときあるじゃないですか。

出力内容をカスタマイズするなら`lib/templates`にファイルを置くだけでいけるのですが、出力処理そのものをカスタマイズするのはどうやるんだろうと思い調べてみた。

## まずは普通にGeneratorを自作してみる

ひとまず適当なRailsアプリ上でGeneratorを作ってみます。

Railsアプリを生成。

```
$ rails new generator-sample
$ cd generator-sample
$ bundle install
```

カスタムGeneratorを生成する。これだけでGeneratorのひな形が出来上がります。

```
$ rails generate generator mofmof
      create  lib/generators/mofmof
      create  lib/generators/mofmof/mofmof_generator.rb
      create  lib/generators/mofmof/USAGE
      create  lib/generators/mofmof/templates
      invoke  test_unit
      create    test/lib/generators/mofmof_generator_test.rb
```

`lib/generators/mofmof/mofmof_generator.rb`編集します。Generatorを実行すると`#copy_initializer_file`が実行されるらしい。

```
class MofmofGenerator < Rails::Generators::NamedBase
  source_root File.expand_path('../templates', __FILE__)

  def copy_initializer_file
    puts "もふもふ => #{file_name}"
  end
end
```

実行結果はこうなる。なるほど。

```
$ rails generate mofmof hoge
もふもふ => hoge
```

## デフォルトのcontroller_generatorを拡張してみる

コントローラーのファイル名の頭に`mofmof_`って勝手につけるGeneratorを作ってみます。

先ほどと同じようにGeneratorのひな形を生成します。

```
$ rails generate generator mofmof_controller
      create  lib/generators/mofmof_controller
      create  lib/generators/mofmof_controller/mofmof_controller_generator.rb
      create  lib/generators/mofmof_controller/USAGE
      create  lib/generators/mofmof_controller/templates
      invoke  test_unit
      create    test/lib/generators/mofmof_controller_generator_test.rb
```

`mofmof_controller_generator.rb`を編集します。

```
require 'rails/generators/rails/controller/controller_generator'

module Rails
  module Generators
    class MofmofControllerGenerator < ControllerGenerator
      source_root File.expand_path("#{base_root}/rails/controller/templates", __FILE__)

      def create_controller_files
        template 'controller.rb', File.join('app/controllers', class_path, "mofmof_#{file_name}_controller.rb")
      end
    end
  end
end
```

Rails標準の`ControllerGenerator`を継承して、`source_root`で標準のテンプレートのパスを指定してあげるのがミソ。

これがないと、`controller.rb`がないよ！って怒られちゃう。

```
Could not find "controller.rb" in any of your source paths. Please invoke Rails::Generators::MofmofControllerGenerator.source_root(PATH) with the PATH containing your templates. Currently you have no source paths.
```

実行結果がこちら。controller名に勝手に`mofmof_`がついてますね。

```
$ r g mofmof_controller moge
      create  app/controllers/mofmof_moge_controller.rb
      invoke  erb
      create    app/views/moge
      invoke  test_unit
      create    test/controllers/moge_controller_test.rb
      invoke  helper
      create    app/helpers/moge_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/moge.coffee
      invoke    scss
      create      app/assets/stylesheets/moge.scss
```

調べていくうちにGeneratorに関していろいろわかった。次は役に立ちそうなGeneratorを作りたい。
