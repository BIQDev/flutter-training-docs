# (U)pdate

Berikutnya kita akan "menjahit" ke REST API kita fungsi (U)pdate

## UI button `onPressed`

Langkah pertama kita adalah mengatur UI kita. Yaitu mem-*bind* tombol edit kita pada *property* `onPressed` dengan fungsi yang akan memanggil UI input.

Buka file `lib/widgets/home/book-list-item.dart`. kemudian ubah/tambah sesuai dengan baris yang ter-*highlight* berikut:

```dart linenums="1" hl_lines="3 4 17-24 41-43"
import 'package:flutter/material.dart';

import 'package:perpus/screens/book-input.dart';

class BookListItem extends StatelessWidget {
  @required
  final String apiHost;
  @required
  final String id;
  @required
  final String title;
  @required
  final String imagePath;

  BookListItem({this.apiHost, this.id, this.title, this.imagePath});

  _update(BuildContext context) {
    Navigator.of(context).pushNamed(
      BookInputScreen.routeName,
      arguments: BookInputScreenArguments(
          id: this.id, title: this.title, imagePath: this.imagePath),
    );
  }

  @override
  Widget build(BuildContext context) {
    return GridTile(
      child: Image.network(
        "${this.apiHost}/perpus-api/${this.imagePath}",
        width: 80,
        height: 80,
        fit: BoxFit.cover,
      ),
      header: Row(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          IconButton(
              splashColor: Colors.orange[100],
              color: Colors.deepOrangeAccent,
              icon: Icon(Icons.edit),
              onPressed: () {
                this._update(context);
              }),
          IconButton(
            color: Colors.redAccent,
            icon: Icon(Icons.delete),
            onPressed: () {},
          ),
        ],
      ),
  ...
```

Pada dasarnya kita memanggil UI yang sama dengan yang kita gunakan untuk (C)reate. Untuk membedakan tampilan input UI nya, kita menggunakan *property* `argument`. *Property* tersebut adalah bagian dari `Navigator.of(context).pushNamed()`.

Penggunaan `argument` tersebut bisa dilihat pada code diatas baris ke-20, yaitu `arguments: BookInputScreenArguments(...)`

Logic nya adalah, jika `title` pada `argument` kosong, maka UI input screen `BookInputScreen` akan memiliki *behavior* mode (C)reate. Sebaliknya, jika `title` pada `argument` berisi string tertentu, maka UI input screen `BookInputScreen` akan pada mode (U)pdate.

Logic tersebut sudah tersedia pada `lib/screens/book-input.dart`. Hanya saja kita perlu sedikit menambahkan code untuk "mengamil `argument`" yang kita kirim dari `arguments: BookInputScreenArguments()` pada code diatas. Mari kita buka file `lib/screens/book-input.dart`, tambah/ubah code seperti pada baris ter-*highlight* berikut:

```dart linenums="85" hl_lines="5"
  ...
  @override
  Widget build(BuildContext context) {
    bool isCreating = context.watch<BookListProvider>().isCreating;
    this._args = ModalRoute.of(context).settings.arguments;
    return Scaffold(
      appBar: AppBar(
        title: this._args == null || this._args.id == null
            ? const Text("Tambah Buku")
            : const Text("Edit Buku"),
      ),
      body: Padding(
```

Sekarang, coba simpan dan jalankan aplikasi android kita, kemudian click/tap icon "edit" pada salah satu buku. Seharusnya "screen" dari input kita akan memiliki title "Edit Buku" pada bagian kiri atas. Bukan "Tambah buku" seperti pada prosedur (C)reate.

