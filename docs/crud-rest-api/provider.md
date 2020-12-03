# Provider
**Provider** digunakan untuk beberapa tujuan. Salah satu kegunaan utamanya adalah untuk CRUD pada project pelatihan ini.

Pada umumnya **provider** disebut juga sebagai *state management*, dimana sebuah aplikasi memiliki satu sumber data untuk semua *user interface*. Kita bisa membayangkan **provider** ini seperti **database** yang berisi satu atau lebih **table**.

Sehingga, pada penggunaan dengan tujuan CRUD, **provider** berfungsi seperti database sementara pada saat terjadi **Read** dari server.

## Latar belakang penggunaan provider
Konsep **provider** pada Flutter, sudah banyak dan populer digunakan pada platform lain, seperti **Redux** pada **RreactJS** atau **NgRx** dan **NgXs** pada **Angular**.

Sebelum konspe *state management* seperti **provider** ini populer, data pada aplikasi antar *user interface* biasanya di *pass* berupa *variable* biasa dengan mengakses module/instance dari *user interface* yang memiliki data terkait. Hal ini akan sangat merepotkan, bayangkan saat kita memiliki sebuah *input dialog*, dari situ kita memerlukan data dari UI lain misal dari panel utama, kita harus memikirkan bagamana mem-*pass* data dari panel utama ke UI *input dialog* tersebut. Terlebih lagi jika pada suatu titik terjadi perubahan data pada panel utama tersebut.

Dibandingkan dengan kosep **provider** ini atau yang umumnya disebut juga dengan **state management**. Data akan dapat diakses pada **satu** instance khusus. Jadi aplikasi kita akan memiliki *single source of truth*, yaitu, semua aplikasi akan memiliki pada sumber *state* yang sama.

## Konsep dasar provider
Konsep dasar **provider**, hampir sama dengan *state management* pada umumnya.

1. Ada satu pusat *state* yang diatur dalam mekanisme *state management*
1. *user interface* atau komponen apa saja yang membutuhkan *state* atau *data* dari **provider**, bisa melakukan

    1. *listen*, dalam hal ini, komponen atau *user interface* terkait akan mengambil satu atau beberapa variable dari **provider**, kemudian akan melakukan *listen*, atau mengawasi perubahan data yang diambil tersebut. Jika terjadi perubahan, maka akan ada prosedur tertentu yang akan dilakukan, misal mengupdate tampilan sesuai perubahan data. Ini adalah fitur yang paling membantu dan sering digunakan. Komponen yang mengunakan, biasanya disebut **listener**
    1. *once time access* atau satukali akses, yang akan mendapatkan hasil data dari **provider** hanya pada saat diambil pada titik tertentu. Misal, hanya di akses pada constructor atau *live cycle* tertentu seperti `didChangeDependencies()`. Jadi tidak akan ada mekanisme *listen*


## Install provider
**Provider** tidak disertakan secara *builtin* pada flutter. Kita harus men-*setup* terpisah.

Untuk itu perlu dilakukan seperti setting package seperti dibawah ini:

1. Buka file `pubspec.yaml`
1. Buat baris baru setelah baris ke-25
1. Tambah code seperti tertera pada baris yang bertanda
   ```dart linenums="23" hl_lines="4"
   dependencies:
     flutter:
       sdk: flutter
     provider: ^4.3.2+2
     image_picker: ^0.6.7+14
     mime: ^0.9.7
   ```
1. Simpan file `pubspec.yaml`, VS Code akan otomatis melakukan **get** untuk package **provider**. Jika tidak, jalankan perintah pada commandline: `flutter pub get`

## Membuat provider
Membuat provider pada flutter cukup sederhana. Ada 2 komponen dasar yang harus dibuat yaitu:

1. Model
    Kita dapat menganalogikan membuat model seperti mendefinisikan *field* atau nama kolom pada table sebuah database. Jadi bisa kita artikan **model** adalah sebuah **definisi**. Model tersebut di definisikan dengan menggunakan `class` biasa dari **dart**.
    
    Model pada dasarnya tidak mandatory, atau tidak wajib. Namun lebih bersifat pada ketergantungan dari *pattern* yang umum digunakan. Karena akan sulit memiliki record yang dishare antara **Provider** dan komponen yang me-*listen*. Hingga ahirnya, umumnya menjadi sebuah keharusan dalam membuat **provider**, kecuali pada kasus tertentu yang didalamnya tidak terdapat *record*, atau type data primitive bawaan dart, seperti integer, string dll.

1. Provider
    Disinilah provider sesungguhnya mulai dibuat. *Signature* umum yang biasanya terdapat pada **provider** adalah sebagai berikut:
    
    1. keyword `with ChangeNotifier`
    
        keyword `with ChangeNotifier` didepan nama class, disebut **mixin** pada **dart** language. Fungsinya hampir seperti *inheritance* class pada **dart** dengan menggunakan keyword **extend**.
        
        Contoh:
        
        ```dart
        class BookListProvider with ChangeNotifier {
        ```
       
       Dengan melakukan **mixin** diatas terhadap class `ChangeNotifier`, provider yang kita definisikan, secara ajaib akan membangun *channel* yang berfungsi untuk memberitahu kepada *listener* atau komponen yang melakukan *listen* pada *state* tertentu. Mengenai *listener* akan kitabahas lebih lanjut nanti.
       
    1. Member variable `List` dengan **generic type**

        Disinilah definisi **Model** diatas berperan. Model yang sudah didefinisikan akan digunakan sebagai *generic type* dari sebuah variable dengan type `List` atau pada pemrograman lain biasa disebut `Array`. Member variable inilah yang nantinya akan digunakan dan di *listen* oleh komponen-komponen lainnya. Data tersebut sering disebut sebagai *state*, karena merupakan variable yang akan berubah2 dan menentukan *state* sebuah aplikasi.
        
        Contoh pendefinisian member variable untuk state dengan generic type:
        ```dart
        List<BookListModel> 
        ```
        
    1. Member variable state bersifat *private*
    
        Ini adalah sebuah pattern wajib. Dengan membuat member variable dari *state* bersifat private, akan mencegah perubahan sembarangan dari luar mekanisme/pattern **provider** itu sendiri
        
        Dari contoh sebelumnya diatas mengenai *generic type*, contoh lengkapnya untuk membuat menjadi private sebagai berikut:
        ```dart
        List<BookListModel> _list = [];
        ```
       
    1. Membuat **getter** untuk mengambil state

        Komponen-komponen yang akan mengambil *state* dari *provider*, harus menggunakan **fungsi khusus** yang di sebut **getter**. Pada pemrograman **dart**, getter didefinisikan dengan menambah *keyword* `get` pada definisi fungsinya. Dengan format: `<type> get <nama_getter>{ return ... }`
        
        Menyambung contoh sebelumnya, getter untuk *state* `_list` diatas menjadi:
        
        ```dart
          List<BookListModel> get list {
            return [...this._list];
          }
        ```
       
       Kenapa return harus ditulis `return [...this._list];` ? karena jika hanya melakukan `return this._list;` *listener* tidak akan mendeteksi saat ada perubahan variable *state* `_list`. Ini adalah sifat alami *dart*, dimana variable akan di *pass* ke komponen lain berupa *pointer* atau alamatnya saja. Dengan me-*return* `return [...this._list];`, dart akan memberikan variable baru sehingga alamat *pointer* akan berubah, dan *listener* akan tahu saat terjadi perubahan variable *state* terkait.
