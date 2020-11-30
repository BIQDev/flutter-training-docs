# Overview
Rencana pengembangan aplikasi

## Layout

Berikut ini layout aplikasi android yang akan kita buat. Pada

### A. Layout halaman utama ( Home )

Halaman utama / home terdiri dari 3 widget seperti pada screenshot berikut.

![Layout](../assets/images/crud/crud-1.0.png){: style="height:auto;width:350px"}

Layout tersebut dibentuk pada code: `lib/screens/home-page.dart`

![Home page](../assets/images/crud/crud-1.1.png)

#### 1. Header
Komponen header pada screenshot diatas adalah sebuah "custom widget" yang di definisikan pada file : `lib/widgets/home/header.dart`

#### 2. Grid view
Pada bagian grid view ini nanti akan tampil daftar buku. Bagian ini juga dibuat dengan custom widget yang didefinisikan pada file: `lib/widgets/home/book-list.dart`

#### 3. Floating button
Bagian terakhir ini ada tombol yang akan menampilkan "screen" baru, yaitu screen "input".

