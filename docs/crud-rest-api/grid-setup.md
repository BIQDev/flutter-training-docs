# Grid setup
Langkah awal, kita akan mulai dengan mensetup pada bagian "Grid". Ini adalah bagian utama yang akan menampilkan daftar buku yang diinput

Pertama, download source code dari: [https://github.com/BIQDev/unisnu-flutter-2020.11/tree/crud-0](https://github.com/BIQDev/unisnu-flutter-2020.11/tree/crud-0)
## GridView.builder()
Flutter memiliki sebuah class yang bernama `GridView`. Class tersebut bertujuan untuk mengatur layout dalam tampilan "grid".

Terdapat berbagai macam metode yang bisa digunakan untuk menginstanisasi class tersebut. Namun untuk tujuan dinamis seperti pada project CRUD ini, dimana jumlah "child" dari grid yang ditampilkan akan berubah-ubah, maka akan lebih tepat untuk menggunakan constructor `GridView.builder()`. Constuctor lain seperti `GridView.count()`, lebih tepat digunakan untuk menampilkan grid "child" yang jumlahnya tetap dan telah ditentukan. Seperti bisa dibaca pada dokumentasi resmi flutter: [https://api.flutter.dev/flutter/widgets/GridView-class.html](https://api.flutter.dev/flutter/widgets/GridView-class.html)

## Grid item ( "grid child" )
Daftar buku pada "Grid view", tiap buku akan ditampilkan dengan sebuah widget yang bernama `Gridtile`. Widget tersebut adalah bawaan dari flutter, yang memang dikhususkan untuk menjadi "child" dari sebuah "Grid view". Meskipun sebenarnya kita buat widget sendiri ( custom widget ).

### Setup asset
Langkah awal, kita buat "mockup" dari widget yang akan menampilkan buku. Sebelumya kita setup dulu "asset" yang kita perlukan untuk "preview". Yaitu gambar buku sebagai bagian icon.

Untuk itu, kita sudah memiliki sebuah file gambar pada `assets/mock/book.png`. Kita tinggal import pada file : `pubspec.yaml` setelah baris 53, sehingga menjadi seperti ini:
```yaml linenums="52" hl_lines="3"
  assets:
    - assets/icon/icon-710x710-android.png
    - assets/mock/book.png
```


### "mock" tampilan buku
Setelah asset di deklarasikan pada file `pubspec.yaml` diatas, gambar tersebut akan kita tampilkan pada "mock" dari `GridTile`.

Buka file `lib/widgets/home/book-list.dart`. Pada baris ke-14 dengan code:

```dart linenums="14"
itemBuilder: (ctx, i) => Text("Dummy"),
```

Kita hapus dan ubah dengan widget `GridTile` sebagai berikut:

```dart linenums="14"
      itemBuilder: (ctx, i) => GridTile(
        child: Image.asset(
          "assets/mock/book.png",
          width: 80,
          height: 80,
          fit: BoxFit.cover,
        ),
        footer: GridTileBar(
          backgroundColor: Colors.black54,
          title: Text("Judulnya"),
          trailing: IconButton(
            splashColor: Colors.red[400],
            icon: Icon(Icons.favorite),
            onPressed: () {},
          ),
        ),
      ),
```

Simpan, dan seharusnya, setelah selesai "hot reload" kita mendapatkan tampilan seperti berikut ini:

![Grid item preview](../assets/images/crud/crud-1.2.png){: style="height:auto;width:350px"}

### Buat "custom widget" grid item
Setelah cukup puas dengan "mockup" pada tampilan Grid Item kita, pindahkan `GridTile()` ke widget terpisah. Hal ini memudahkan kita untuk mengembangkan Grid Item tersebut lebih detail secara terpisah dan substansial.

1. Buat file `lib/widgets/home/book-list-item.dart`
1. Pindahkan `GridTile()` dari `lib/widgets/home/book-list.dart` line 14, pindahkan ke file baru diatas. Sehingga, file `lib/widgets/home/book-list-item.dart` akan menjadi seperti dibawah ini:

    ```dart linenums="1"
    import 'package:flutter/material.dart';
    
    class BookListItem extends StatelessWidget {
      @override
      Widget build(BuildContext context) {
        return GridTile(
          child: Image.asset(
            "assets/mock/book.png",
            width: 80,
            height: 80,
            fit: BoxFit.cover,
          ),
          footer: GridTileBar(
            backgroundColor: Colors.black54,
            title: Text("Judulnya"),
            trailing: IconButton(
              splashColor: Colors.red[400],
              icon: Icon(Icons.favorite),
              onPressed: () {},
            ),
          ),
        );
      }
    }
    
    ```

1. Pada file `lib/widgets/home/book-list.dart` line 2, Tambahkan:

    ```dart linenums="1" hl_lines="2"
    import 'package:flutter/material.dart';
    import 'package:perpus/widgets/home/book-list-item.dart';
    
    class BookList extends StatefulWidget {
    ...
    ```
   Kemudian line 15 menjadi
   ```dart linenums="15"
         itemBuilder: (ctx, i) => BookListItem(),
   ```

### Grid item detail
Pada bagian ini kita akan memberikan sedikit detail pada "grid item" kita. Terutama untuk tujuan navigasi yang berkaitan dengan CRUD, yaitu pada bagian *update* dan *delete*

Beralih kembali ke file `lib/widgets/home/book-list-item.dart` buat baris baru setelah baris ke-12. Pada baris ke-13, masukkan kode berikut:

```dart linenums="13"
      header: Row(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          IconButton(
            splashColor: Colors.orange[100],
            color: Colors.deepOrangeAccent,
            icon: Icon(Icons.edit),
            onPressed: () {},
          ),
          IconButton(
            color: Colors.redAccent,
            icon: Icon(Icons.delete),
            onPressed: () {},
          ),
        ],
      ),
``` 

Akan muncul 2 tombol pada bagian *header* dari "grid item" seperti pada screenshot berikut:

![Grid item header](../assets/images/crud/crud-1.3.png){: style="width: 350px; height: auto;"}

Saat ini tombolnya masih belum memiliki fungsi apa pun, segera kita akan mem-*bind* fungsi *update* dan *delete* ke tombol tersebut.
