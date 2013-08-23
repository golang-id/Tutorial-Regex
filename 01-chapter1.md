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
panic, nilai balik kedua berupa error ditiadakan. Hal ini memudahkan penyambungan `MustCall` dengan
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

Menemukan satu backslash '\': Harus di-_esape_ dua kali di regex dan sekali di string.

~~~go
r, err := regexp.Compile(`C:\\\\`)
if r.MatchString("Working on drive C:\\") == true {
	fmt.Println("Cocok.")
} else {
	fmt.Println("Tidak cocok.")
}
~~~
