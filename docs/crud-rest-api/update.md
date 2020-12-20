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

Logic tersebut sudah tersedia pada `lib/screens/book-input.dart`. Hanya saja kita perlu sedikit menambahkan code untuk "mengambil `argument`" yang kita kirim dari `arguments: BookInputScreenArguments()` pada code diatas. Mari kita buka file `lib/screens/book-input.dart`, tambah/ubah code seperti pada baris ter-*highlight* berikut:

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
  ...
```

Sekarang, coba simpan dan jalankan aplikasi android kita, kemudian click/tap icon "edit" pada salah satu buku. Seharusnya "screen" dari input kita akan memiliki title "Edit Buku" pada bagian kiri atas. Bukan "Tambah buku" seperti pada prosedur (C)reate.

## UI/UX screen update

Kita akan menerapkan beberapa *rules* atau aturan untuk membuat UI/UX kita terasa nyaman pada halaman UI (U)pdate sebagai berikut:

1. *Current value*, atau nilai data saat ini yang sedang diedit, yang akan kita tampilkan pada halaman UI (U)pdate adalah judul buku atau *title*. 
1. Halaman input tidak akan menampilkan gambar saat ini
1. Jika user ingin mengganti gambar, dan memilih gambar, maka gambar baru akan ditampilkan pada form input

### 1. Current value / nilai data saat ini

Pertama kita akan mengeset input kita dengan data yang sedang diedit. Untuk itu kita perlu mengeset field "title" pada input form kita. Akan tetapi, kita tidak akan melakukan set data dari *lifecycle* `build()` yang biasa kita buat dengan code `Widget build(BuildContext context)`. Karena mengeset data dari *lifecycle* `build()`, bisa berdampak pada *recursive* calls, atau pemanggilan fungsi yang berulang tanpa akhir.

Oleh karena itu kita akan menggunakan *lifecycle* `didChangeDependencies()`, yang mana akan dipanggil sebelum terjadinya *lifecycle* `build`. Untuk itu, buka file `lib/screens/book-input.dart` kemudian ubah/tambah sesuai baris yang ter-*highlight* berikut:

```dart linenums="1" hl_lines="27 46-59"
import 'package:flutter/material.dart';
import 'dart:io';
import 'package:image_picker/image_picker.dart';
import 'package:provider/provider.dart';

import 'package:perpus/models/booklist_model.dart';
import 'package:perpus/providers/booklist_provider.dart';

class BookInputScreenArguments {
  final String id;
  final String title;
  final String imagePath;
  BookInputScreenArguments({this.id, this.title, this.imagePath});
}

class BookInputScreen extends StatefulWidget {
  static const routeName = "/book-add";

  @override
  _BookInputScreenState createState() => _BookInputScreenState();
}

class _BookInputScreenState extends State<BookInputScreen> {
  String _title;
  bool _inputIsValid = false;
  BookInputScreenArguments _args;
  bool _isInitialized; // Berfungsi untuk mencegah pemanggilan ganda

  File _image;
  final picker = ImagePicker();

  Future _pickImage() async {
    final pickedFile = await picker.getImage(source: ImageSource.gallery);

    setState(() {
      if (pickedFile != null) {
        _image = File(pickedFile.path);

        this._setInputValid();
      } else {
        print('No image selected.');
      }
    });
  }

  @override
  void didChangeDependencies() {
    if (this._isInitialized == null || !this._isInitialized) {
      this._args = ModalRoute.of(context).settings.arguments;
      if (this._args != null && this._args.id != null) {
        this._title = this._args.title;
      }

      this._setInputValid();
      this._isInitialized = true;
    }
    super.didChangeDependencies();
  }

  void _setInputValid() {
    this._inputIsValid =
        this._title != null && this._title != "" && this._image != null;
  }
  ...
```

Pada code diatas, variable `_isInitialized` berfungsi untuk memastikan pengesetan input "title" hanya terjadi satu kali. Karena *lifecycle* `didChangeDependencies()` akan dipanggil beberapa kali pada proses "inisialisasi" sebuah *widget*.

Simpan perubahan dan jalankan aplikasi. Seharusnya saat menekan tombol edit, field input "title" akan otomatis terisi sesuai nilai sekarang dari buku yang kita edit.
