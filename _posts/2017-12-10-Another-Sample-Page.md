---
title: Connection WriteUp - HackMyVM
published: true
---

Text can be **bold**, _italic_, ~~strikethrough~~ or `keyword`.

[HackMyVM - Connection](https://hackmyvm.eu/machines/machine.php?vm=Connection).

Este es un writeup que hice en 2021 de nivel fácil. En esta máquina enumeramos un servicio SMB en el que nos aprovecharemos para subir nuestra propia shell. Para la escalada utilizaremos `gdb` con permisos SUID para escalar a root.


# [](#header-1)Enumeración
Con el comando `sudo  arp-scan -l | grep -i "PCS" | awk '{print $1}'` obtenemos directamente la ip haciendo un escaneo ARP en nuestra red local filtrando por la nomenclatura PCS utilizada por VirtualBox para definir en red sus dispositivos virtualizados.
![image](https://github.com/cerodah/blog/assets/82907557/e9750184-0568-409e-9899-4e1edb19c3c5)




## [](#header-2)Nmap
A través de `nmap -p22,80,139,445 -sV -sC $IP -oN nmap` vemos 4 puertos en los que hay 3 servicios corriendo. 
> ![image](https://github.com/cerodah/blog/assets/82907557/aac8dc27-16df-4f3a-bc4c-c90212968599)


Pasamos directamente a mirar el servidor samba para investigar en el.
> ![image](https://github.com/cerodah/blog/assets/82907557/433574b8-0f2c-47a1-9d40-34bd54be5906)

Listamos los recursos compartidos en el servidor y vemos un recurso llamado share.
> ![image](https://github.com/cerodah/blog/assets/82907557/272800d0-6900-416d-9a96-306759ed9d7f)

### [](#header-1)Explotación


```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### [](#header-4)Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### [](#header-5)Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### [](#header-6)Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![](https://assets-cdn.github.com/images/icons/emoji/octocat.png)

### Large image

![](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
