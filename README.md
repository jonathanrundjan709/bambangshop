# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. Menurut saya, untuk kondisi BambangShop saat ini `struct Subscriber` tunggal sudah cukup dan belum wajib dibuat sebagai `trait`. Pada pola Observer klasik, `Subscriber` dijadikan interface karena ada kemungkinan banyak tipe observer dengan perilaku `update()` yang berbeda-beda. Di tahap ini, data subscriber yang kita simpan hanya `url` dan `name`, lalu semua subscriber diperlakukan dengan cara yang sama: menerima notifikasi HTTP ke endpoint mereka. Karena belum ada variasi perilaku antarsubscriber, menambah `trait` justru menambah kompleksitas tanpa manfaat nyata. `Trait` baru terasa perlu jika nanti ada beberapa jenis subscriber dengan mekanisme update berbeda, misalnya ada yang menerima webhook, ada yang menyimpan log lokal, atau ada yang memerlukan transformasi payload khusus.

2. Menurut saya, `Vec` saja kurang ideal karena `id` pada `Program` dan `url` pada `Subscriber` dimaksudkan unik. Jika memakai `Vec`, pengecekan duplikasi, pencarian, dan penghapusan harus dilakukan dengan iterasi linear, sehingga kompleksitasnya menjadi `O(n)`. Untuk repository yang sering melakukan operasi tambah, cari, dan hapus berdasarkan key unik, struktur map seperti `DashMap` lebih tepat karena akses berbasis key lebih langsung dan intent desainnya lebih jelas: memang ada identitas unik yang dipakai sebagai indeks. Jadi, untuk kebutuhan sekarang, penggunaan map bukan sekadar optimasi kecil, tetapi lebih cocok dengan aturan domain bahwa setiap entitas punya identifier unik.

3. `DashMap` dan Singleton menyelesaikan masalah yang berbeda, jadi Singleton saja tidak menggantikan kebutuhan `DashMap`. Singleton hanya mengatur bahwa instance penyimpanan bersifat tunggal dan dibagikan secara global. Namun, Singleton tidak otomatis membuat akses ke data di dalamnya aman untuk banyak thread. Di Rust, karena aplikasi web Rocket dapat menangani banyak request secara paralel, kita tetap memerlukan struktur data yang thread-safe saat singleton itu diakses bersama-sama. Dalam kasus ini, `lazy_static` sudah membantu membuat satu instance global `SUBSCRIBERS`, sedangkan `DashMap` memastikan operasi baca/tulis ke koleksi tersebut aman dan efisien pada lingkungan concurrent. Jadi, implementasi sekarang sebenarnya adalah kombinasi keduanya: pola Singleton untuk instance global, dan `DashMap` sebagai mekanisme penyimpanan thread-safe di dalam singleton itu.

#### Reflection Publisher-2
1. Menurut saya, pemisahan `Service` dan `Repository` dari `Model` dibutuhkan agar setiap komponen punya tanggung jawab yang jelas. `Model` sebaiknya fokus merepresentasikan data dan aturan dasar objek, `Repository` fokus pada penyimpanan dan akses data, sedangkan `Service` menangani orchestration atau business logic antarobjek. Ini sejalan dengan Single Responsibility Principle karena alasan perubahan tiap lapisan jadi berbeda. Misalnya, perubahan mekanisme penyimpanan subscriber tidak seharusnya memaksa perubahan pada struct `Subscriber`, dan perubahan alur subscribe/unsubscribe tidak seharusnya membuat repository tahu soal detail HTTP atau controller. Dengan pemisahan ini, kode juga lebih mudah diuji, dipelihara, dan dikembangkan tanpa membuat satu struct menjadi terlalu gemuk.

2. Jika semua hal hanya diletakkan di `Model`, maka setiap model akan makin saling bergantung dan kompleksitasnya cepat naik. `Program` bisa saja harus tahu cara mencari subscriber, membuat notification, dan menghapus data terkait. `Subscriber` bisa ikut memuat logika penyimpanan dirinya sendiri sekaligus cara menerima atau mengirim notifikasi. `Notification` juga bisa terdorong untuk tahu detail `Program` dan `Subscriber`. Akibatnya, coupling antar model menjadi tinggi dan satu perubahan kecil dapat merambat ke banyak file. Model-model itu akan berubah dari sekadar representasi domain menjadi objek serba bisa yang menangani data, koordinasi, dan persistence sekaligus. Dalam jangka panjang, pendekatan itu membuat kode lebih sulit dibaca, lebih sulit dites secara terisolasi, dan lebih rawan menghasilkan duplikasi logika.

3. Ya, saya cukup tertarik dengan Postman karena tool ini membantu menguji endpoint tanpa harus menulis client sementara. Untuk pekerjaan saat ini, Postman memudahkan saya mengirim request `POST` dan melihat apakah body JSON, status code, dan response sudah sesuai, terutama untuk endpoint subscribe dan unsubscribe. Fitur yang menurut saya paling berguna adalah collections untuk menyimpan daftar request, environment variables agar URL host bisa diganti cepat, serta body editor JSON yang memudahkan percobaan payload. Saya juga tertarik dengan fitur test scripts dan runner karena itu bisa dipakai untuk regression testing ringan ketika jumlah endpoint makin banyak. Untuk group project maupun proyek software engineering lain, Postman berguna bukan hanya sebagai alat kirim request manual, tetapi juga sebagai dokumentasi API yang hidup dan mudah dibagikan ke anggota tim.

#### Reflection Publisher-3
1. Pada tutorial ini, menurut saya kita memakai variasi **Push model**. Publisher tidak menunggu subscriber mengambil data sendiri, tetapi langsung mengirim payload notifikasi ke setiap subscriber ketika ada event seperti `CREATED`, `PROMOTION`, atau `DELETED`. Ini terlihat dari adanya method `notify()` yang menyiapkan data lalu memanggil `update()` milik setiap subscriber melalui HTTP POST. Artinya, informasi perubahan didorong dari publisher ke subscriber secara aktif.

2. Jika pada kasus ini kita memakai **Pull model**, ada beberapa keuntungan tetapi juga trade-off yang jelas. Keuntungannya, payload notifikasi dari publisher bisa lebih kecil karena publisher cukup memberi sinyal bahwa ada perubahan, lalu subscriber mengambil detail produk sendiri bila perlu. Ini membuat publisher tidak perlu menentukan semua isi data yang dibutuhkan subscriber. Namun, kekurangannya cukup besar untuk tutorial ini: subscriber jadi harus tahu cara meminta detail ke publisher, sehingga coupling ke API publisher justru meningkat. Selain itu, setiap event bisa memicu request tambahan dari banyak subscriber, yang berarti jumlah network call total bisa naik. Untuk kasus BambangShop yang notifikasinya sederhana dan payload-nya kecil, pendekatan push lebih praktis karena subscriber langsung menerima informasi yang relevan tanpa langkah tambahan.

3. Jika kita memutuskan tidak memakai multi-threading dalam proses notifikasi, maka publisher akan mengirim notifikasi ke subscriber satu per satu secara berurutan. Dampaknya, waktu respons endpoint seperti create, publish, atau delete akan ikut menunggu seluruh proses pengiriman selesai. Jika ada satu subscriber yang lambat atau bermasalah, request dari client ke publisher juga akan ikut tertahan lebih lama. Dalam aplikasi web, ini dapat menurunkan throughput dan membuat pengalaman pengguna terasa lambat, terutama saat jumlah subscriber bertambah. Dengan multi-threading, proses notifikasi bisa dijalankan paralel sehingga request utama dapat selesai lebih cepat dan bottleneck dari satu subscriber tidak terlalu menghambat subscriber lain.
