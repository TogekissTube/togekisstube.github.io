---
layout: post
title: Creando un sistema operativo desde cero (parte 1)
---

En este articulo, te enseñare a crear un sistema operativo DESDE cero y funcional, bueno empezemos.

Recursos:

https://wiki.osdev.org/Main_Page
https://github.com/chipsetx/Simple-Kernel-in-C-and-Assembly/tree/master

----


Vamos a crear el directorio del OS, en mi caso TogekissOS:

mkdir TogekissOS (en mi caso)

Ahora entremos en el directorio y creemos el directorio src.

cd TogekissOS (en mi caso)
mkdir src
cd src


Antes de programar, instalemos las dependencias

Ubuntu/Debian:

sudo apt install build-essential nasm qemu-system-i386

Fedora:

sudo dnf install @development-tools nasm qemu-system-x86

Arch:

sudo pacman -S gcc binutils nasm qemu


Bueno vamos a empezar a programar, el primer archivo es un boot.asm, y no, no es un bootloader.

Es un archivo de inicio para cargar el kernel, (que programaremos en C), y posiblemente, sea el archivo mas importante del OS:

Codigo: 

bits 32
section .text

      ; multiboot

      align 4
      dd 0x1BADB002	
      dd 0x00
      dd - (0x1BADB002 + 0x00)	


global start
extern main

start:

    cli
    call main
    hlt


Puedes usar cualquier IDE como Visual Studio code o Nano (no es un IDE pero sirve)

Ahora, vamos a crear el archivo de kernel (se llama kernel.c)

Codigo:

/* kernel.c */

/* Definición de la función principal del kernel */
void main() {
    /* Puntero a la dirección de memoria de video */
    char* video_memory = (char*) 0xB8000;

    /* Mensaje de saludo */
    char* message = "Hello world!\n";

    /* Color celeste en formato de video de texto */
    char color = 0x03;

    /* Pintar el mensaje en la memoria de video con color celeste */
    for(int i = 0; message[i] != '\0'; ++i) {
        /* Carácter ASCII */
        video_memory[i*2] = message[i];
        /* Formato de color */
        video_memory[i*2+1] = color;
    }
}


Ahora el penultimo archivo, el que sirve para crear el ELF del kernel, el linker.ld:

Codigo:

ENTRY(_start)

SECTIONS {
    . = 1M;

    .text ALIGN(4K) : {
        *(.text)
    }

    .rodata ALIGN(4K) : {
        *(.rodata)
    }

    .data ALIGN(4K) : {
        *(.data)
    }

    .bss ALIGN(4K) : {
        *(.bss)
    }

    /DISCARD/ : {
        *(.comment)
        *(.eh_frame)
    }
}


Ahora salgamos del directorio src.

cd ..

Ahora para compilar el OS tenemos que usar un archivo propio llamado build.sh:

Codigo:

cd src
nasm -f elf32 boot.asm -o boot.o
gcc -m32 -c kernel.c -o kernel.o
ld -m elf_i386 -T linker.ld -o kernel.elf boot.o kernel.o
qemu-system-i386 -kernel kernel.elf

Ahora pon estos comandos:

chmod +x build.sh
./build.sh

Y terminamos y si todo salio bien, saldra esto

<img src="{{ site.baseurl }}/images/parte1.png" style="width: 400px;"/>


Adios!!!!
