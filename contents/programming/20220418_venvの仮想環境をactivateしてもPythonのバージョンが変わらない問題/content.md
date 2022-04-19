## 環境

- macOS
- Python3.9.5

## 発生した問題

pyenvでPythonをインストール後、仮想環境を作成した。その際、仮想環境をactivateして、指定したバージョンのPythonを起動できることを確認した。

数日後、同様に仮想環境をactivateしPythonを起動しようとすると、当該バージョンではなく、システムのPythonが起動してしまった。

## 原因

仮想環境を作成してから次に使うまでの間に、仮想環境関連のファイルが置いてあるディレクトリの構造が変わっていた。それに伴い仮想環境へのパスがずれていた。

## 詳細と再現方法

何もしていない状態でPythonコマンドを叩くとシステムのPythonが起動する。

```sh
python -V
>>> Python 2.7.16

python3 -V
>>> Python 3.8.2
```

仮想環境を作成・activateし、Pythonのバージョンが正しく表示されることを確認する。

```sh
~/.pyenv/versions/3.9.5/bin/python -m venv .test_venv
source .test_venv/bin/activate

python -V
>>> Python 3.9.5
```

仮想環境から一旦出て、親ディレクトリ名を変更する。
```sh
deactivate

cd ..
mv parent parent_renamed
```

もう一度作業ディレクトリに戻り、仮想環境に入りPythonのバージョンを確認する。

```sh
source .test_venv/bin/activate

python -V
>>> Python 2.7.16
```

指定したはずの3.9.5ではなく、システムの2.7.16が表示される。

ここで、venvの起動スクリプト`.test_venv/bin/activate`を確認すると

```
...

VIRTUAL_ENV="/path/to/parent/.test_venv"

...

```

となっており、古いままのディレクトリ名になっている。つまり、親ディレクトリ名を変更したにも関わらず、パスが古いままであるため正しい仮想環境へのパスが指定できておらず、切り替えがうまくいっていない。

これを新しいディレクトリ名に書き換えれば、仮想環境へのパスを正しく指してくれるようになる。

```
...

VIRTUAL_ENV="/path/to/parent_renamed/.test_venv"

...

```
仮想環境で指定したバージョンのPythonが起動することを確認する。

```sh
python -V
>>> Python 3.9.5
```

## 感想

venvが仮想環境を切り替える仕組みを意識したことがなかったのですが、環境変数を一時的に書き換えるという原始的な方法（とはいえ他に方法があるのかはわからないが）でやっているのだなと勉強になりました。
