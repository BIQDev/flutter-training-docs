# (C)reate then (R)ead

Saat ini, jika kita menginput data baru yaitu (C)reate, maka saat kembali kehalaman "Home", tampilan daftar buku atau "book list" kita tidak ada perubahan. Meskipun sebenarnya data sudah terkirim ke server.

Untuk itu, kita perlu memasang prosedur (R)ead setelah (C)reate berhasil.

## Tangkap hasil kembalian input

Pertama yang harus kita lakukan adalah menangkap nilai kembalian dari "screen" input kita. Hal tersebut bisa didapat dari titik dimana kita memicu/membuka "screen" dari input. Yaitu pada file `lib/screens/home-page.dart`. Buka file tersebut, dan lihat pada baris yang ter-highlight berikut:

```dart linenums="5" hl_lines="5-9"
...
class MyHomePage extends StatelessWidget {
  MyHomePage({Key key}) : super(key: key);

  _newBook(BuildContext addButtonContext) async {
    Navigator.of(addButtonContext).pushNamed(
      BookInputScreen.routeName,
    );
  }

  @override
  Widget build(BuildContext context) {
  ...
```

Pada fungsi `_newBook()`, saat ini kita hanya membuka `BookInputScreen` tanpa menangkap hasil kembaliannya. Sekarang kita harus menangkap nilai kembalian dari `BookInputScreen` pada sebuah variable seperti berikut:

```dart linenums="5" hl_lines="6-8"
...
class MyHomePage extends StatelessWidget {
  MyHomePage({Key key}) : super(key: key);

  _newBook(BuildContext addButtonContext) async {
    final result = await Navigator.of(addButtonContext).pushNamed(
      BookInputScreen.routeName,
    );
  }

  @override
  Widget build(BuildContext context) {
  ...
```

Pada code diatas, baris ke-10, kita menerima nilai kembalian dari `BookInputScreen` ke variable `result`. Kemudian nanti variable `result` akan kita gunakan untuk menentukan jika hasil dari `result == 200`, maka akan melakukan (R)ead. `200` adalah standard **HTTP code** untuk response sukses.

Bagian yang menyebabkan `BookInputScreen` mendapatkan response code `200` adalah dari file `lib/screens/book-input.dart` pada baris yang ter-highlight berikut:

```dart linenums="69" hl_lines="3"
...
    if (submitRes["statusCode"] != null && submitRes["statusCode"] == 200) {
      Navigator.pop(context, submitRes["statusCode"]);
    } else {
      Scaffold.of(submitContext)
        ..removeCurrentSnackBar()
        ..showSnackBar(
          SnackBar(
            content: Text(
                "Error \n- Status: ${submitRes["statusCode"]} \n- Message: ${submitRes["message"]}"),
            duration: Duration(seconds: 5),
            backgroundColor: Colors.redAccent.shade400,
          ),
        );
    }
...
```

Fungsi `Navigator.pop()` adalah fungsi untuk menutup "screen" dan kembali ke screen berikutnya yaitu screen "home".

## Gunakan variable `result`

Setelah kita buat variable `result` seperti diatas, dan tahu dari mana nilai kembalian yang didapatkan, saatnya kita gunakan variable tersebut untuk memicu (R)ead jika proses (C)reate selesai. Kembali ke file `lib/screens/home-page.dart`, tepat dibawah definisi variable `result`, kita buat sebagaiberikut:

```dart linenums="1" hl_lines="2 3 7 16-27"
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'package:perpus/screens/book-input.dart';
import 'package:perpus/widgets/home/book-list.dart';
import 'package:perpus/widgets/home/header.dart';
import 'package:perpus/providers/booklist_provider.dart';

class MyHomePage extends StatelessWidget {
  MyHomePage({Key key}) : super(key: key);

  _newBook(BuildContext addButtonContext) async {
    final result = await Navigator.of(addButtonContext).pushNamed(
      BookInputScreen.routeName,
    );

    if (result == 200) {
      Scaffold.of(addButtonContext)
        ..removeCurrentSnackBar()
        ..showSnackBar(
          SnackBar(
            content: Text("Berhasil menambah buku"),
            backgroundColor: Colors.greenAccent.shade700,
          ),
        );
      addButtonContext.read<BookListProvider>().read(addButtonContext);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
...
```

Simpan perubahan diatas, dan untuk memastikan perubahan terload, kita tekan tombol restart ![restart](../assets/images/icons/restart.png){: style="height:25px; width:auto;"}. Coba untuk melakukan input data buku baru, kemudian simpan. Jika anda kembali ke halaman "home" dan input data baru tampil, maka anda berhasil :tada:.

Jika tidak, harap periksa kembali, mungkin ada yang salah :smile:.

## Tentang `addButtonContext`

Mungkin ada yang bertanya-tanya tentang `addButtonContext` ?. Kita perlu `addButtonContext` karena `context` pada `Widget build(BuildContext context)` tidak bisa digunakan untuk melakukan **scaffold** pada screen lain seperti yang kita perlukan pada `BookInputScreen`. Untuk itu kita perlu membuat context baru yang akan digunakan `scaffold()`. Pada kasus ini kita meletakkan pada file `lib/screens/home-page.dart`, perhatikan *code snippet* berikut mulai dari line 42 sampai 52:

```dart linenums="41"
      ...
      floatingActionButton: Builder(
        builder: (BuildContext addButtonContext) {
          return FloatingActionButton(
            onPressed: () {
              this._newBook(addButtonContext);
            },
            tooltip: 'New Product',
            child: Icon(Icons.add),
          );
        },
      ),
      ...
```

Pada *code snippet* diatas, kita membuat **context** baru dengan menggunakan `Builder()`, dengan nama context baru `addButtonContext`.

Jika kita tidak melakukan hal diatas dan memaksa menggunakan **context** dari `Widget build(BuildContext context)`, maka kita akan mendapatkan error sebagai berikut:

> FlutterError (Scaffold.of() called with a context that does not contain a Scaffold.
> No Scaffold ancestor could be found starting from the context that was passed to Scaffold.of(). This usually happens when the context provided is from the same StatefulWidget as that whose build function actually creates the Scaffold widget being sought.
> There are several ways to avoid this problem. The simplest is to use a Builder to get a context that is "under" the Scaffold. For an example of this, please see the documentation for Scaffold.of():
> https://api.flutter.dev/flutter/material/Scaffold/of.html
> A more efficient solution is to split your build function into several widgets. This introduces a new context from which you can obtain the Scaffold. In this solution, you would have an outer widget that creates the Scaffold populated by instances of your new inner widgets, and then in these inner widgets you would use Scaffold.of().
> A less elegant but more expedient solution is assign a GlobalKey to the Scaffold, then use the key.currentState property to obtain the ScaffoldState rather than using the Scaffold.of() function.
> The context used was:
> MyHomePage)
