Bagian 1: Dasar
===============

## Pencocokan Sederhana

Anda ingin mengetahui apakah sebuah string sesuai dengan regular expression. Fungsi `MatchString` akan
mengembalikan `true` jika argumen string sesuai dengan regular expression yang Anda buat dengan `Compile`.

~~~go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	r, err := regexp.Compile(`Hello`)
	if err != nil {
		fmt.Printf("There is a problem with your regexp.\n")
		return
	}

	// Akan mencetak 'Cocok'
	if r.MatchString("Hello Regular Expression.") == true {
		fmt.Println("Cocok")
	} else {
		fmt.Println("Tidak cocok")
	}
}
~~~

`Compile` merupakan jantung dari paket `regexp`. Setiap regular expression harus disiapkan dengan
`Compile` atau fungsi sejenisnya `MustCompile`. Fungsi `MustCompile` sama dengan `Compile`, tapi
akan panic jika regular expression tidak dapat dikompil. Karena error di `MustCompile` berujung ke
panic, nilai balik kedua berupa error ditiadakan. Hal ini memudahkan penyambungan `MustCompile` dengan
fungsi match yang Anda inginkan, seperti ditunjukkan dibawah: (Tapi Anda perlu menghindari kompilasi
berulang dalam _looping_ dengan alasan performa)

~~~go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	if regexp.MustCompile(`Hello`).MatchString("Hello Regular Expression.") == true {
		fmt.Println("Cocok") // Akan mencetak 'Cocok'
	} else {
		fmt.Println("Tidak cocok")
	}
}
~~~

Berikut adalah regex yang salah:

~~~go
var myre = regexp.MustCompile(`d(+`)
~~~

yang akan menghasilkan:

~~~text
panic: regexp: Compile(`\d(+`): error parsing regexp: missing argument to repetition operator: `+`

goroutine 1 [running]:
regexp.MustCompile(0x4de620, 0x4, 0x4148e8)
    go/src/pkg/regexp/regexp.go:207 +0x13f
~~~

Fungsi `Compile` mengembalikan error di nilai balik kedua. Pada tutorial ini saya akan mengabaikan error,
karena regex yang disajikan disini sempurna ;-). Mungkin Anda boleh saja mengikuti cara ini jika regexp-nya literal, tapi kalau regexp berasal dari input Anda sebaiknya mengecek nilai error.

Tutorial selanjutnya akan mengabaikan nilai error agar lebih ringkas.

Regular expression berikut tidak akan cocok:

~~~go
r, err := regexp.Compile(`Hxllo`)
// Akan mencetak 'false'
fmt.Printf("%v", r.MatchString("Hello Regular Expression."))
~~~

TODO: CompilePOSIX/MustCompilePOSIX.

## Karakter class

Karakter class '\w' merepresentasikan karakter dari class [A-Za-z0-9_], mnemonic: 'word'.

~~~go
r, err := regexp.Compile(`H\wllo`)
// Akan mencetak 'true'
fmt.Printf("%v", r.MatchString("Hello Regular Expression."))
~~~

Karakter class '\d' merepresentasikan digit numerik.

~~~go
r, err := regexp.Compile(`\d`)
// Akan mencetak 'true'
fmt.Printf("%v", r.MatchString("Seven times seven is 49"))
// Akan mencetak 'false'
fmt.Printf("%v", r.MatchString("Seven times seven is forty-nine."))
~~~

Karakter class '\s' merepresentasikan _whitespace_: TAB, SPACE, CR, LF. Atau lebih tepatnya [\t\n\f\r].

~~~go
r, err := regexp.Compile(`\s`)
// Akan mencetak 'true':
fmt.Printf("%v", r.MatchString("/home/bill/My Documents"))
~~~

Karakter class dapat dinegasikan menggunakan huruf besar '\D', '\S', '\W'. Jadi, '\D' merupakan karaketer yang bukan '\d'.

~~~go
r, err := regexp.Compile(`\S`) // Bukan whitespace
// Akan mencetak 'true', jelas ada non-whitespace disini:
fmt.Printf("%v", r.MatchString("/home/bill/My Documents"))
~~~

Mengecek _filename_ sebagai validitas (Catatan: Lingkup filesystem/encoding yang berbeda akan menyebabkan masalah yang berbeda. Apakah Anda tahu '\n' merupakan karater yang sah untuk _filename_ untuk POSIX? [D. Paper Wheeler mengenai Posix filenames](http://www.dwheeler.com/essays/fixing-unix-linux-filenames.html).)

~~~go
// FIXME This is nonsense.
r, err := regexp.Compile(`\W`) // Karakter yang bukan '\w'.
// Akan mencetak 'false', tidak ada karakter non-word disini:
fmt.Printf("%v", r.MatchString("my_extraordinary_but_valid_filename.txt"))
~~~

## Mana yang cocok?

Fungsi `FindString` berfungsi menemukan sebuah string. Jika Anda menggunakan literal string, hasilnya jelas string itu sendiri. Hasilnya akan lebih menarik jika Anda menggunakan pola dan class.

~~~go
r, err := regexp.Compile(`Hello`)
// Akan mencetak 'Hello'
fmt.Printf(r.FindString("Hello Regular Expression. Hullo again."))
~~~

Ketika `FindString` tidak menemukan string yang sesuai dengan regular expression, maka string kosong akan dikembalikan. Hati-hati, string kosong bisa saja hasil yang valid.

~~~go
r, err := regexp.Compile(`Hxllo`)
// Akan mencetak string kosong
fmt.Printf(r.FindString("Hello Regular Expresion."))
~~~

`FindString` langsung mengembalikan hasil pertama yang cocok. Jika Anda menginginkan hasil-cocok lebih dari satu gunakan `FindAllString` seperti dijelaskan di bawah.

## Karakter Spesial

Karakter '.' cocok dengan semua karakter.

~~~go
// Akan mencetak 'cat'
r, err := regexp.Compile(`.at`)
fmt.Printf(r.FindString("The cat sat on the mat."))
~~~

'cat' merupakan hasil-cocok pertama.

~~~go
// more dot.
s := "Nobody expects the Spanish inquisition."
//           __ __     __
r, err := regexp.Compile(`e.`)
res := r.FindAllString(s, -1) // negative: semual hasil-cocok
// Mencetak [ex ec e ]. Hasil terakhir merupakan 'e' dan spasi.
fmt.Printf("%v", res)
res = r.FindAllString(s, 2) // temukan 2 atau kurang hasil-cocok
// Prints [ex ec].
fmt.Printf("%v", res)
~~~

## Karakter Spesial Literal

Menemukan satu backslash '\': Harus di-_esape_ di regex dan di string.

~~~go
r, err := regexp.Compile(`C:\\`)
if r.MatchString("Working on drive C:\\") == true {
	fmt.Println("Cocok.") // <---
} else {
	fmt.Println("Tidak cocok.")
}
~~~

Menemukan literal dot:

~~~go
r, err := regexp.Compile(`\.`)
if r.MatchString("Short.") {
	fmt.Println("Has a dot")  // <---
} else {
	fmt.Println("Has no dot.")
}
~~~

Karakter spesial lainnya yang perlu ditulis dengan cara serupa: `+*?()|[]{}^$`.

Menemukan literal simbol dollar:

~~~go
r, err := regexp.Compile(`\$`)
if len(r.FindString("He paind $150 for that software.")) != 0 {
	fmt.Println("Found $-symbol") // <---
} else {
	fmt.Println("No $$$.")
}
~~~

## Pengulangan Sederhana

Fungsi `FindAllString` mengembalikan sebuah array berisi semua string yang cocok. `FindAllString`
memiliki dua argumen, sebuah string dan `int` yang menyatakan maksimal hasil-cocok untuk dikembalikan.
Jika Anda menginginkan semua hasil-cocok gunakan '-1'.

Kata merupakan barisan karater bertipe `\w`. Simbol `+` menyatakan pengulangan. Menemukan kata:

~~~go
s := "Eenie meenie miny moe."
r, err := regexp.Compile(`\w+`)
res := r.FindAllString(s, -1)
// Mencetak [Eenie meenie miny moe]
fmt.Printf("%v", res)
~~~

Tidak seperti penggunaan _wildcard_ di _command-line_ untuk pencocokan filename,
`*` tidak mensimbolkan 'karakter apapun' melainkan pengulangan terhadap karakter
sebelumnya (atau grup). Kalau `+` membutuhkan setidaknya satu kemunculan untuk
karakter di awal simbol, `*` tetap terevaluasi jika tidak ada kemunculan. Hal ini
dapat menimbulkan hasil yang tidak sesuai harapan.

~~~go
s := "Firstname Lastname"
r, err := regexp.Compile(`\w+\s\w+)
res := r.FindString(s)
// Mencetak Firstname Lastname
fmt.Printf("%v", res)
~~~

Tapi jika `s` dari user input, mungkin saja bisa ada dua spasi:

~~~go
s := "Firstname  Lastname"
r, err := regexp.Compile(`\w+\s\w+)
res := r.FindString(s)
// Mencetak string kosong
fmt.Printf("%v", res)
~~~

Untuk memperbolehkan setidaknya satu spasi, gunakan `\s+`:

~~~go
s := "Firstname  Lastname"
r, err := regexp.Compile(`\w+\s+\w+`)
res := r.FindString(s)
// Mencetak Firstname  Lastname
fmt.Printf("%v", res)
~~~

Jika Anda membaca berkas INI, mungkin Anda perlu toleran terhadap spasi disekitar '='.

~~~go
s := "Key=Value"
r, err := regexp.Compile(`\w+=\w+`)
res := r.FindAllString(s, -1)
// Mencetak Key=Value
fmt.Printf("%v", res)
~~~

Sekarang tambahkan spasi disekitar '='.

~~~go
s := "Key = Value"
r, err := regexp.Compile(`\w+=\w+`)
res := r.FindAllString(s, -1)
// GAGAL, mencetak string kosong karena \w tidak menangkap spasi.
fmt.Printf("%v", res)
~~~

Untuk menangkap sejumlah spasi (termasuk 0) gunakan `\s*`:

~~~go
s := "Key = Value"
r, err := regexp.Compile(`\w+\s*=\s*\w+`)
res := r.FindAllString(s, -1)
fmt.Printf("%v", res)
~~~

Ada beberapa lagi pola yang ditulis dengan `?` yang tersedia di regexp Go.

## Anchor dan Boundaries

Simbol sisipan `^` menandakan 'awal dari baris'.

~~~go
s := "Never say never."
r, err1 := regexp.Compile(`^N`)     // Apakah 'N' ada di awal?
fmt.Printf("%v ", r.MatchString(s)) // true
t, err2 := regexp.Compile(`^n`)     // Apakah 'n' ada di awal?
fmt.Printf("%v ", t.MatchString(s)) // false
~~~

Simbol dolar `$` menandakan 'akhir dari baris'.

~~~go
s := "All is well that ends well"
r, err := regexp.Compile(`well$`)
fmt.Printf("%v ", r.MatchString(s)) // true

r, err = regexp.Compile(`well`)
fmt.Printf("%v ", r.MatchString(s)) // true, tapi hasilnya merupakan
                                    // dari 'well' pertama
r, err = regexp.Compile(`ends$`)
fmt.Printf("%v ", r.MatchString(s)) // false, tidak muncul di akhir baris.
~~~

Dapat dilihat 'well' cocok dengan pola. Untuk mengetahui dimana regexp
menemukan hasil-cocok dapat dilihat dengan `FindStringIndex`. Fungsi
`FindStringIndex` mengembalikan array dengan dua elemen. Elemen pertama
(di indeks 0) merupakan indeks pada string dimana regular expression
cocok. Elemen kedua adalah indeks setelah regexp berakhir.

~~~go
s := "All is well that ends well"
//    012345678901234567890123456
//              1         2
r, err := regexp.Compile(`well$`)
fmt.Printf("%v", r.FindStringIndex(s)) // Mencetak [22 26]

r, err = regexp.Compile(`well`)
fmt.Printf("%v", r.MatchString(s))  // true, tapi hasilnya merupakan
                                    // dari 'well' pertama
r.Printf("%v", r.FindStringIndex(s)) // Mencetak [7 11], mulai dari 7 dan berakhir sebelum 11.

r, err = regexp.Compile(`ends$`)
fmt.Printf("%v", r.MatchString(s)) // false, tidak muncul di akhir baris.
~~~

Anda dapat batas kata dengan `\b`. Fungsi `FindAllStringIndex` menangkap semua yg cocok
di dalam array.

~~~go
s := "How much wood would a woodchuck chuck in Hollywood?"
//    012345678901234567890123456789012345678901234567890
//              10        20        30        40        50
//             -1--         -2--                    -3--
// Menemukan semua kata *yang diawali* dengan wood
r, err := regexp.Compile(`\bwood`)            //    1       2
fmt.Printf("%v", r.FindAllStringIndex(s, -1)) // [[9 13] [22 26]]

// Menemukan semua kata *yang diakhiri* dengan wood
r, err = regexp.Compile(`wood\b`)             //    1       3
fmt.Printf("%v", r.FindAllStringIndex(s, -1)) // [[9 13] [46 50]]

// Menemukan kata yang *diawali* dan *diakhiri* dengan wood
r, err = regexp.Compile(`\bwood\b`)           //    1
fmt.Printf("%v", r.FindAllStringIndex(s, -1)) // [[9 13]]
~~~

## Karater class

Disamping literak karakter, set (atau class) karakter dapat digunakan disetiap posisi.
Misalnya `[uio]` merupakan "karakter class". Setiap karakter di dalam kurung kotak akan
cocok dengan regexp. Regexp berikut akan cocok dengan 'Hullo', 'Hillo', dan 'Hollo':

~~~go
r, err := regexp.Compile(`H[uio]llo`)
// Akan mencetak 'Hullo'.
fmt.Printf(r.FindString("Hello Regular Expression. Hullo again."))
~~~

Karakter class yang dinegasikan membalikan hasil-cocock dari class. Contoh berikut akan cocok
dengan semua string 'H.llo', dimana dot bukan 'o', 'i' atau 'u', tapi cocok dengan "Hallo", bahkan
"H9llo":

~~~go
r, err := regexp.Compile(`H[^uio]llo`)
fmt.Printf("%v", r.MatchString("Hillo")) // false
fmt.Printf("%v", r.MatchString("Hallo")) // true
fmt.Printf("%v", r.MatchString("H9llo")) // true
~~~

## Karakter Class POSIX
Pustaka Go regexp mengimplementasikan karakter class POSIX. Berikut merupakan alias yang lebih mudah
dibaca untuk class yang sering digunakan ([https://re2.googlecode.com/hg/doc/syntax.htm](https://re2.googlecode.com/hg/doc/syntax.htm)):

~~~text
[:alnum:]   alphanumeric (≡ [0-9A-Za-z])
[:alpha:]   alphabetic (≡ [A-Za-z])
[:ascii:]   ASCII (≡ [\x00-\x7F])
[:blank:]   blank (≡ [\t ])
[:cntrl:]   control (≡ [\x00-\x1F\x7F])
[:digit:]   digits (≡ [0-9])
[:graph:]   graphical (≡ [!-~] == [A-Za-z0-9!"#$%&'()*+,\-./:;<=>?@[\\\]^_`{|}~])
[:lower:]   lower case (≡ [a-z])
[:print:]   printable (≡ [ -~] == [ [:graph:]])
[:punct:]   punctuation (≡ [!-/:-@[-`{-~])
[:space:]   whitespace (≡ [\t\n\v\f\r ])
[:upper:]   upper case (≡ [A-Z])
[:word:]    word characters (≡ [0-9A-Za-z_])
[:xdigit:]  hex digit (≡ [0-9A-Fa-f])
~~~

Perlu dicatat bahwa Anda perlu mengurung karakter class ASCII didalam `[]`. Saat
kita membahas alphabet maka yang dimaksud adalah 26 huruf (65-90), tidak termasuk
huruf dengan tanda diakritik.

Contoh: Menemukan deretan huruf kecil, karakter tanda baca, spasi/tab (blank) dan digit:

~~~go
r, err := regexp.Compile(`[[:lower:]][[:punct:]][[:blank:]][[:digit:]]`)
if r.MatchString("Fred: 123456789") == true {
    //               ----
	fmt.Printf("Cocok")
} else {
	fmt.Printf("Tidak cocok")
}
~~~

Saya tidak pernah menggunakannya karena lebih banyak yang diketik, tapi cocok digunakan di
proyek dengan banyak pengembang dimana tidak semuanya memahami regular expression seperti
Anda.

## Unicode Class

Unicode tersusun atas blok, dikelompokan berdasarkan topik atau bahasa. Pada bab ini saya hanya memberikan
beberapa contoh, karena tidak memungkinkan untuk membahas semuanya. Silahkan lihat di [daftar lengkap unicode dari re2 engine](https://code.google.com/p/re2/wiki/Syntax).

### Contoh: Greek

Kita mulai dengan contoh sederhana dari blok kode Greek.

~~~go
r, err := regexp.Compile(`\p{Greek}`)

if r.MatchString("This is all Γςεεκ to me.") == true {
	fmt.Printf("Cocok") // Cetak 'Cocok'
} else {
	fmt.Printf("Tidak cocok")
}
~~~

Di Windows-1252 codepage ada mu, tapi tidak memenuhi kualifikasi, karena `\p{Greek}` hanya
menjangkau U+0370..U+03FF (http://en.wikipedia.org/wiki/Greek_and_Coptic).

~~~go
if r.MatchString("the µ is right before ¶") == true {
	fmt.Printf("Cocok")
} else {
	fmt.Printf("Tidak cocok") // Akan mencetak 'Tidak cocok'
}
~~~

Beberapa huruf keren lainnya dari codepage Greek dan Coptik yang memenuhi syarat sebagai 'Greek'
meskipun sebenarnya adalah Coptic.

~~~go
if r.MatchString("ϵ϶ϓϔϕϖϗϘϙϚϛϜ") == true {
	fmt.Printf("Cocok") // Akan mencetak 'cocok'
} else {
	fmt.Printf("Tidak cocok") 
}
~~~

### Contoh: Braille

Anda harus menggunakan font yang mendukung [Braille](http://en.wikipedia.org/wiki/Braille). Saya ragu ini berguna kecuali
digunakan untuk printer Braille, tapi berikut merupakan contohnya:

~~~go
r2, err := regexp.Compile(`\p{Braille}`)
if r2.MatchString("This is all ⢓⢔⢕⢖⢗⢘⢙⢚⢛ to me.") == true {
	fmt.Printf("Match ") // Will print 'Match'
} else {
	fmt.Printf("No match ")
}
~~~

### Contoh: Cherokee

Anda harus menggunakan font yang mendukung Cherokee (misal Code2000). Kisah mengenai skrip Cherokee layak
[untuk dibaca](http://en.wikipedia.org/wiki/Cherokee#Language_and_writing_system).

~~~go
r3, err := regexp.Compile(`\p{Cherokee}`)
if r3.MatchString("This is all ᏯᏰᏱᏲᏳᏴ to me.") == true {
	fmt.Printf("Cocok") // Akan mencetak 'Cocok'
} else {
	fmt.Printf("Tidak cocok")
}
~~~

## Alternatif

Anda dapat menyediakan alternatif hasil-cocok dengan simbol pipe '|'. Jika Anda menginginkan hasil-cocok
hanya bagian dari regular expression, gunakan tanda kurung tutup-buka untuk _grouping_.

~~~go
r, err1 := regexp.Compile(`Jim|Tim`)
fmt.Printf("%v", r.MatchString("Dickie, Tom and Jim")) // true
fmt.Printf("%v", r.MatchString("Jimmy, John and Jim")) // true

t, err2 := regexp.Compile(`Santa Clara|Santa Barbara`)
s := "Clara was from Santa Barbara and Barbara was from Santa Clara"
//                   -------------                      -----------
fmt.Printf("%v", t.FindAllStringIndex(s, -1))
// [[15 28] [50 61]]

u, err3 := regexp.Compile(`Santa (Clara|Barbara)`) // Equivalent
v := "Clara was from Santa Barbara and Barbara was from Santa Clara"
//                   -------------                      -----------
fmt.Printf("%v", t.FindAllStringIndex(v, -1))
// [[15 28] [50 61]]
~~~
