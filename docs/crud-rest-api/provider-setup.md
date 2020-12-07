# Provider setup
Pada bagian ini, kita akan membuat 2 buah provider, yaitu:

1. Setting

    Provider ini digunakan untuk mendefinisikan setting. Dalam hal ini hanya untuk menentukan `<host>` dan `<user_name>` pada REST API sesuai dengan yang telah dijelaskan pada halaman overview [REST API](overview.md#rest-api)
    
    Sebenarnya untuk setting tidak harus selalu menggunakan **provider**, dalam project ini hanya untuk kita jadikan contoh simple pembuatan **provider** yang sederhana. Namun tidak ada salahnya pada project real menggunakan **provider** untuk menyimpan setting. Karena bisa diakses darimana saja.

1. Book List

    Daftar buku yang didapat dari CRUD khususnya bagian (R)ead, akan disimpan pada provider ini. Selain *state* yang menyimpan daftar buku, juga akan ada state lain seperti:
    
    1. `_isReading`, yaitu state yang akan bernilai `true` saat proses (R)ead terjadi. Dan bernilai `false` saat proses (R)ead selesai. Hal ini bertujuan untuk mengatur tampilan *UI*. Dimana  saat bernilai `true`, *UI* akan menampilkan *loading indicator*, sedangkan saat bernilai `false`, *UI* akan menampilkan daftar buku.
    2. `_isCreating` hampir sama dengan `_isReading`, hanya saja ini digunakan pada proses (C)reate.
    
## Langkah awal

Pertama, buat 2 directory:

1. models

    Untuk menyimpan file-file definisi model yang akan digunakan provider
    
2. providers

    Untuk menyimpan file-file definisi provider itu sendiri
    
## Provider untuk Setting

Kita mulai dengan provider untuk setting.

Seperti yang tertulis di [Membuat provider](provider.md#membuat-provider), secara umum kita perlu membuat **model** dan **provider**.

### SettingModel

Buat file `lib/models/setting_model.dart`, masukkan code berikut:

```dart linenums="1"
class SettingModel {
  final String apiHost;
  final String userName;

  SettingModel({this.apiHost, this.userName});
}

```

ada 2 variable yang akan digunakan pada model ini. Yaitu `apiHost` dan `userName`. Seperti dijelaskan sebelumnya, variable tersebut akan digunakan untuk menentukan URL dari REST API.

### SettingProvider

Buat file `lib/providers/setting_provider.dart`, kemudian copy code berikut kedalamnya:

```dart linenums="1"
import 'package:flutter/material.dart';
import 'package:perpus/models/setting_model.dart';

class SettingProvider with ChangeNotifier {
  SettingModel _setting = SettingModel(
    apiHost: "https://perpus-api.biqdev.com",
    userName: "nama-anda",
  );

  SettingModel get setting {
    return _setting;
  }
}
```

- **Catatan penting**, untuk setting ini, **provider** yang akan digunakan adalah type *static*. Tidak seperti **provider** yang nanti akan kita pakai untuk daftar buku( *book list*) yang bersifat *dynamic*. Karena **provider** untuk *book list* akan berubah-ubah sesuai data dari server REST API
- seperti dijelaskan sebelumnya `with ChangeNotifier` adalah sebuah metode **mixin** agar class yang kita buat dapat berkomunikasi dengan **listener** nya.
- `apiHost` diatas sudah running dan siap pakai. Jika ingin serve sendiri, REST API tersebut bisa setting sendiri dengan mendapatkan source dari : [https://github.com/BIQDev/perpus-api](https://github.com/BIQDev/perpus-api)
- Ganti `userName` dengan nama anda, usahakan unik dan tidak dipakai user lain agar tidak terjadi konflik. Untuk simpelnya, kita tidak ada proses registrasi :smile:
- `SettingModel get setting{}`, seperti dijelaskan sebelumnya adalah **getter** dari variable state yaitu `_setting` ( private variable) 

## "Jahit" provider
Sebelum kita membuat provider lain, marilah kita "jahit" dulu satu **provider** yang baru saja kita buat. Semua provider-provider yang akan kita buat nantinya, akan menjadi **"root"**, atau "akar"/awal dari "widget tree"/"pohon widget" kita. Agar widget-widget lain berada dibawah dari **provider**, sehingga semua widget pada apps kita, secara architecture bisa mengambil data dari **provider** yang kita buat. Berikut ini step nya:

- buka file `lib/main.dart`
- import **provider**, buat line baru setelah line ke-1 ( line ke-2 ).
    ```dart linenums="2"
    import 'package:provider/provider.dart';
    ```
- import **provider** seting yang sudah kita buat. Buat line baru setelah line ke-5:
    ```dart linenums="6"
    import 'package:perpus/providers/setting_provider.dart';
    ```
- lihat pada line ke-15, kita akan membuat **widget** `MaterialApp` yang awalnya sebagai **root widget** dari aplikasi kita, akan menjadi **child** dari **provider**
- hapus line ke-16 s/d line ke-25 yaitu:
    ```dart linenums="16"
        return MaterialApp(
          title: 'Perpus',
          theme: ThemeData(
            primarySwatch: Colors.blue,
          ),
          home: MyHomePage(),
          routes: {
            BookInputScreen.routeName: (ctx) => BookInputScreen(),
          },
        );
    ```
- ganti dengan code berikut:
    ```dart linenums="16"
        return MultiProvider(
          providers: [
            ChangeNotifierProvider(create: (_) => SettingProvider()),
          ],
          child: MaterialApp(
            title: 'Perpus',
            theme: ThemeData(
              primarySwatch: Colors.blue,
            ),
            home: MyHomePage(),
            routes: {
              BookInputScreen.routeName: (ctx) => BookInputScreen(),
            },
          ),
        );
    ```
    Code diatas, pada dasarnya memindahkan `MaterialApp(...)` yang sebelumnya sebagi "root" menjadi "child".

 
