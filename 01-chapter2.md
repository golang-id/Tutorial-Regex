Bagian 2: Tahap Lanjut
======================

## Grup

Terkadang Anda ingin hasil-cocok terhadap string, tapi hanya tertarik pada slice tertentu saja. Di bab sebelumnya,
kita selalu melihat ke _semua_ hasil-cocok string.

~~~go
// [[cat] [sat] [mat]]
re, err := regexp.Compile(`.at`)
res := re.FindAllStringSubmatch("The cat sat on the mat.", -1)
fmt.Printf("%v", res)
~~~

Tanda kurung tutup-buka dapat menangkap bagian string yang Anda tertarik bukan semua regex.

~~~go
// [[cat c] [sat s] [mat m]]
re, err := regexp.Compile(`(.)at`) // ingin mengetahui apa yang di depan 'at'
res := re.FindAllStringSubmatch("The cat sat on the mat.", -1)
fmt.Printf("%v", res)
~~~

Anda dapat menggunakan lebih dari satu grup.

~~~go
// Mencetak [[ex e x] [ec e c] [e  e  ]]
s := "Nobody expects the Spanish inquisition."
re1, err := regexp.Compile(`(e)(.)`) // Menyiapkan regex
result_slice := re1.FindAllStringSubmatch(s, -1)
fmt.Printf("%v", result_slice)
~~~

Fungsi `FindAllStringSubmatch` mengembalikkan array dimana untuk setiap hasil-cocok merupakan array dengan
hasil-cocok sepenuhnya di elemen pertama dan isi grup di elemen berikutnya.

Jika terdapat grup opsional yang tidak muncul di string, hasil array akan berupa string kosong, dengan kata
lain, jumlah elemen akan selalu sama dengan dengan jumlah grup ditambah satu.

~~~go
s := "Mr. Leonard Spock"
re1, err := regexp.Compile(`(Mr)(s)?\. (\w+) (\w+)`)
result := re1.FindStringSubmatch(s)

for k, v := range result {
	fmt.Printf("%d. %s\n", k, v)
}

// Mencetak
// 0. Mr. Leonard Spock
// 1. Mr
// 2.
// 3. Leonard
// 4. Spock
~~~

Grup yang timpang-tindih secara parsial tidak dapat dilakukan. Misal jika regexp pertama yang diinginkan cocok dengan 'expects the'
dan lainnya cocok dengan 'the Spanish' maka tanda kurung-tutup akan diinterpretasi secara berbeda. Moto-nya: Yang terakhir terbuka
ditutup pertama kali. Tanda kurung buka-tutup yang dibuka untuk 'the' ditutup setelah 'the'.

~~~go
s := "Nobody expects the Spanish inquisition."
re1, err := regexp.Compile(`(expects (...) Spanish)`)
// Wanted regex1          --------------
// Wanted regex2                   --------------
result := re1.FindStringSubmatch(s)

for k, v := range result {
	fmt.Printf("%d: %s\n", k, v)
}
// 0. expects the Spanish
// 1. expects the Spanish
// 2. the

...

## Hasil-cocok dengan nama

Akan ditulis