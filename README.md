# Receh Explained

## Javascript Stack

adalah tempat penampung (wadah) dari setiap fungsi yang diberikan saat process event loop terjadi

![stack-2-1.png](https://learnersbucket.com/wp-content/uploads/2018/12/stack-2-1.png)

+ **Call Stack**
adalah sebuah proses yang befungsi memanggil setiap fungsi yang akan di eksekusi, yang ditampung kedalam sebuah stack yang diberikan saat process event loops terjadi, yang dimana process nya itu
sendiri menggunakan konsep Last In First Out (LIFO), yang dimana yang posisinya terakhir itu yang pertama kali di eksekusi dan akan keluar dari stack ketika process nya itu sudah terpenuhi, jika stack sudah menjadi empty maka eventloop akan memasukan fungsi selanjutnya (jika ada) kedalam stack.

![SlxsrqM.png](https://i.imgur.com/SlxsrqM.png)

+ **Maximum Call Stack**

bisa terjadi dikarenakan fungsi yang sedang berada didalam sebuah stack tidak pernah kunjung selesai terpenuhi dan fungsi yang berada didalam sebuah stack mencetak nilai berulang kali tanpa henti melebihi size ukuran stack default yang telah ditentukan maka terjadilah error maximum callstack, dikarenakan masing - masing resource juga mempunyai batasan ukuran stacknya tersendiri, kenapa harus ada maximum callstack? dikarenakan jika tidak ada proses error maximum callstack, dipastikan aplikasi yang kita buat akan crash, dikarenakan terlalu banyak penggunakan resource cpu/ram dari bad code yang kita tulis, walaupun itu keliahatannya seperti code pada umumnya yang sering kita tulis, jadi intinya maximum callstack dibuat untuk memberhentikan process yang berulang kali mencetak nilai melebihi stack size default, check [disini](https://stackoverflow.com/questions/7826992/browser-javascript-stack-size-limit]) untuk batasan stack untuk browser atau googling dengan keyword limited stack in windows,linux or programming language.

```js
// example one
var items = [];
[].push.apply(items, new Array(1000000));

// example two
var i = 100000
function print() {
	i -= 1
	console.log(i)
	if (i != 0) {
		print()
	}
}
print()

// example three
function data(n) {
  console.log(n)
  data(n - 1)
}

data(10)

```

![5264759717888000.png](https://www.educative.io/cdn-cgi/image/f=auto,fit=contain,w=600/api/edpresso/shot/5624475006533632/image/5264759717888000.png)


## Non Blocking (Asynchronous)

adalah sebuah process yang dimana process nya tidak saling tunggu satu sama lain sampai, sebagai contoh misalkan saya mempunya 3 fungsi yang dimana fungsi ke 2 sudah di eksekusi terlebih dahulu begitu juga dengan fungsi ke 3, tanpa perlu harus menunggu fungsi ke 1 tersebut selesai di eksekusi, contoh seperti code dibawah ini, oh iya bagi yang belum tau callback itu juga termasuk asyncronous.

```js
const fs = require('fs')
const { Worker } = require('worker_threads')
const axios = require('axios')

// example one
fs.readFile('.prettierrc', (err, data) => {
  if (!err) console.log('file 1: ' + data)
})

fs.readFile('.prettierrc', (err, data) => {
  if (!err) console.log('file 2: ' + data)
})

fs.readFile('.prettierrc', (err, data) => {
  if (!err) console.log('file 3: ' + data)
})

// example two
  const worker1 = new Worker('./worker1.js', { workerData: { url: 'https://jsonplaceholder.typicode.com/users' } })
  const worker2 = new Worker('./worker2.js', { workerData: { url: 'https://jsonplaceholder.typicode.com/users' } })
  const worker3 = new Worker('./worker3.js', { workerData: { url: 'https://jsonplaceholder.typicode.com/users' } })

  worker1.once('message', (result) => {
    console.log('first execute: ', result[0].id)
  })

  worker2.once('message', (result) => {
    console.log('second execute: ', result[1].id)
  })

  worker3.once('message', (result) => {
    console.log('third execute: ', result[2].id)
  })
```

## Blocking (Synchronous)

adalah sebuah process yang dimana process nya itu saling tunggu satu sama lain sampai process sebelumnya itu selesai (sequential), sebagai contoh misalkan saya mempunyai 3 fungsi yang dimana fungsi ke 2 harus menunggu fungsi ke 1 selesai terlebih dahulu, begitu juga dengan fungsi ke 3 yang dimana harus menunggu fungsi ke 2 selesai terlebih dahulu  baru bisa mencetak hasil nilainya.

```js
const fs = require('fs')

// example one
const file1 = fs.readFileSync('./.prettierrc')
const file2 = fs.readFileSync('./.prettierrc')
const file3 = fs.readFileSync('./.prettierrc')

console.log('file1 : ' + file1)
console.log('file2 : ' + file2)
console.log('file3 : ' + file3)

// example two
function test1() {
  console.log('testing 1')
}

function test2() {
  console.log('testing 2')
}

function test3() {
  console.log('testing 3')
}

test1()
test2()
test3()
```

## Concurrency

adalah sebuah proses yang menjalankan sebuah fungsi secara bersamaan dalam waktu yang sama, tetapi tidak menutup kemungkinan concurrency juga bisa tidak menjalankan fungsi secara bersamaan dalam waktu yang sama, yang dimana proses kerja dari concurrency itu sendiri tidak terlalu membebani process kinerja dari cpu dikarenakan concurrency menggunakan single thread untuk menjalan sebuah fungsi secara bersamaan, concurrency memang sekilas mirip dengan parallel tetapi ini adalah sesuatu hal yang berbeda, contoh misalkan saya mempunyai 6 buah fungsi yang berbeda tetapi saya ingin menjalankannya secara concurrency bukan parallel, yang berarti bisa saja dari 6 fungsi tersebut saya menjalankan yang pertama itu 3 fungsi step selanjutnya saya menjalankan 3 jadi total yang dijalankan adalah 6, beda halnya dengan parallel dimana ke 6 fungsi tersebut akan dijalankan secara bersamaan dalam waktu yang sama.

```js
import PQueue from 'p-queue'
import axios from 'axios'

const queue = new PQueue({ concurrency: 3 })

setInterval(() => {
  ;(async () => {
    const data = await queue.add(() => axios.get('https://jsonplaceholder.typicode.com/users'))
    console.log('first execute: ', data.data[0].id)
  })()
  ;(async () => {
    const data = await queue.add(() => axios.get('https://jsonplaceholder.typicode.com/users'))
    console.log('second execute: ', data.data[0].id)
  })()
  ;(async () => {
    const data = await queue.add(() => axios.get('https://jsonplaceholder.typicode.com/users'))
    console.log('third execute: ', data.data[0].id)
  })()
  ;(async () => {
    const data = await queue.add(() => axios.get('https://jsonplaceholder.typicode.com/users'))
    console.log('four execute: ', data.data[0].id)
  })()
  ;(async () => {
    const data = await queue.add(() => axios.get('https://jsonplaceholder.typicode.com/users'))
    console.log('five execute: ', data.data[0].id)
  })()
  ;(async () => {
    const data = await queue.add(() => axios.get('https://jsonplaceholder.typicode.com/users'))
    console.log('six execute: ', data.data[0].id)
  })()
}, 1000)
```

![Untitled.jpg](https://techdifferences.com/wp-content/uploads/2017/12/Untitled.jpg)

# Parallel

# Iterable

# Non Iterable

# Immutable

# Mutable

# Recursive

# Closure

# Garbage Collection

# Topic

# Queue

# Big O Notation

# Object oriented Programming

# Design Pattern

# Caching

# Database

# SQL

# NoSQL

# NodeJS

# Indexing
