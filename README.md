# Apresentação ROP

## Sumário

- [x] [Utilidade do ataque](Apresenta%C3%A7%C3%A3o%20ROP.md#Utilidade-do-ataque)

- [x] [Endereços canônicos e não canônicos](Apresenta%C3%A7%C3%A3o%20ROP.md#Endere%C3%A7os-can%C3%B4nicos-e-n%C3%A3o-can%C3%B4nicos)

- [x] [Link estático](Apresenta%C3%A7%C3%A3o%20ROP.md#Link-est%C3%A1tico)

- [x] [Link dinâmico](Apresenta%C3%A7%C3%A3o%20ROP.md#Link-din%C3%A2mico)

    - [x] [Immediate Binding](Apresenta%C3%A7%C3%A3o%20ROP.md#Immediate-Binding)

    - [x] [Lazzy Binding](Apresenta%C3%A7%C3%A3o%20ROP.md#Lazy-Binding)

- [x] [PLT (Procedure Linkage Table)](Apresenta%C3%A7%C3%A3o%20ROP.md#PLT-%28Procedure-Linkage-Table%29)

- [x] [GOT (Global Offset Table)](Apresenta%C3%A7%C3%A3o%20ROP.md#GOT-%28Global-Offset-Table%29)

- [x] [BSS](Apresenta%C3%A7%C3%A3o%20ROP.md#BSS-%28Block-Started-by-Symbol%29)

- [x] [RELRO](Apresenta%C3%A7%C3%A3o%20ROP.md#RELRO)

- [x] [VDSO (Virtual Dynamic Shared Object)](Apresenta%C3%A7%C3%A3o%20ROP.md#VDSO-%28Virtual-Dynamic-Shared-Object%29)

- [x] [ASLR (Address Space Layout Randomization)](Apresenta%C3%A7%C3%A3o%20ROP.md#ASLR-%28Address-Space-Layout-Randomization%29)

- [x] [Stack Canary](Apresenta%C3%A7%C3%A3o%20ROP.md#Stack-Canary)

- [x] [NX (No eXecute)](Apresenta%C3%A7%C3%A3o%20ROP.md#NX-%28No-eXecute%29)

- [x] [PIE (Position Independent Executable)](Apresenta%C3%A7%C3%A3o%20ROP.md#PIE-%28Position-Independent-Executable%29)

- [x]  [BOF (Buffer Overflow)](Apresenta%C3%A7%C3%A3o%20ROP.md#BOF-%28Buffer-Overflow%29)

    - [x] [Shellcode](Apresenta%C3%A7%C3%A3o%20ROP.md#Shellcode)

    - [x] [ROP (Return Oriented Programming)](Apresenta%C3%A7%C3%A3o%20ROP.md#ROP-%28Return-Oriented-Programming%29%0D)

    - [x] [ROP Gadget](Apresenta%C3%A7%C3%A3o%20ROP.md#ROP-Gedget%0D)

-

## Conceitos necessário para o bom entendimento de ROP



### Utilidade do ataque



ROP é um tipo de ataque utilizado como alternativa ao buffer overflow quando a stack está protegida para **execução** e com **code signing**.



***



### Endereços canônicos e não canônicos



Endereços canônicos são uma denominação dos endereços virtuais de memória da forma  presente na arquitetura de processadores  x86-64 geralmente na forma seguinte forma `0x0000000000000000 - 0x0000ffffffffffff`  ou `0x1111000000000000 - 0x1111ffffffffffff`.

Os endereços de memória onde certa quantia de seus n bits até o mais significativo é composto por zeros ou um podem ser tratados por certas fabricantes de processadores como endereços canônicos de memória, fazendo assim com que um registrador possa trabalahar com uma quantidade menor de bits. Os endereços que não atendem a esse padrão são chamados de não canônicos.

Esse endreços são importantes quando é necessário sobreescrever o valor de um registrador de alguma forma, geralmente utilizando [BOF](Apresenta%C3%A7%C3%A3o%20ROP.md#BOF-%28Buffer-Overflow%29), já que esses registadores podem trabalahar somente com endereços canônicos.



***



### Link estático



Um link estático é uma forma de compilação do binário onde as bibliotecas incluidas no código fonte tem seu código inserido no próprio binário fazendo assim com que não seja necessário utilizar uma função para associara bibliotecas externas ao executável e consequentemente que não seja nessário uma [PLT](Apresenta%C3%A7%C3%A3o%20ROP.md#PLT-%28Procedure-Linkage-Table%29) já que todos os simbolos são associados no momento da compilação do binário.



***



### Link dinâmico



Um link estático é uma forma de compilação do binário onde as bibliotecas incluidas no código fonte tem seu código referênciado em um arquivo externo que será associado ao binário em seu tempo de execução conforme a função é necessária, para isso é necessário a utilização de uma [PLT](Apresenta%C3%A7%C3%A3o%20ROP.md#PLT-%28Procedure-Linkage-Table%29) caso o link dinâmico seja feito com [Lazy Binding](Apresenta%C3%A7%C3%A3o%20ROP.md#Lazy-Binding).



#### Immediate Binding



Immediate Binding é uma forma de link dinâmico onde os simbolos são associados na [GOT](Apresenta%C3%A7%C3%A3o%20ROP.md#GOT-%28Global-Offset-Table%29) assim que o processo é carregado na memória não sendo necessário a utilização de uma [PLT](Apresenta%C3%A7%C3%A3o%20ROP.md#PLT-%28Procedure-Linkage-Table%29) já que os simbolos são associados imediatemente após o início da execução do binário.



#### Lazy Binding



Immediate Binding é uma forma de link dinâmico onde os simbolos são associados na [GOT](Apresenta%C3%A7%C3%A3o%20ROP.md#GOT-%28Global-Offset-Table%29) conforme são chamados no tempo de execução, esse processo é feito com auxílio de uma [PLT](Apresenta%C3%A7%C3%A3o%20ROP.md#PLT-%28Procedure-Linkage-Table%29).



***



### PLT (Procedure Linkage Table)



É uma região de memória do binário maracada somente para leitura onde são guardados todos os simbolos (funções) que precisam ser resolvidos. A PLT é responsável por chamar a função  que ira associar o seu endereço na biblioteca em que está a uma entrada da [GOT](Apresenta%C3%A7%C3%A3o%20ROP.md#GOT-%28Global-Offset-Table%29).



***



### GOT (Global Offset Table)



A GOT é uma  região da memória do binário onde são guardados os endereços das funções associadas dinâmicamente, esse endereço é guardado após a primeira chamada da função  para evitar que o programa precise pesquisar pela função na biblioteca em que ela pertence toda vez que a função for chamada. Antes da primeira chamada da função a GOT aponta para uma entrada na [PLT](Apresenta%C3%A7%C3%A3o%20ROP.md#PLT-%28Procedure-Linkage-Table%29).



***



### BSS (Block Started by Symbol)



É uma região de memória responsável por alocar memória para variáveis alocadas estaticamente no programa que ainda não foram inicializadas.



***



### RELRO



É uma proteção que altera a posição e protege a escrita de algumas áreas do binário.

No RELRO parcial a área da [GOT](Apresenta%C3%A7%C3%A3o%20ROP.md#GOT-%28Global-Offset-Table%29) é forçada a estar antes do [BSS](Apresenta%C3%A7%C3%A3o%20ROP.md#BSS-%28Block-Started-by-Symbol%29) na memória, eleminando assim o risco de BOF em varáveis globais sobrescrevendo a [GOT](Apresenta%C3%A7%C3%A3o%20ROP.md#GOT-%28Global-Offset-Table%29).

Já no RELRO completo toda a [GOT](Apresenta%C3%A7%C3%A3o%20ROP.md#GOT-%28Global-Offset-Table%29) é marcada como somente para escrita o que inviabiliza a sobescrita dos endereços de suas funções ou um **ROP gadget** que o atacante pretendesse executar.



***



### VDSO (Virtual Dynamic Shared Object)



VDSO é uma biblioteca compartilhada que o kerenel automaticamente mapeia no binário. Essa biblioteca tarta algumas chamadas mais que o sistema faz ao kernel para que bibliotecas com a libc não precisem se preocupar com a compatibilidade de código a nível de kernel.



***



### ASLR (Address Space Layout Randomization)



ASLR é uma configuração a nível de kernel, no linux geralmente pode ser acessado através do caminho `/proc/sys/kernel/randomize_va_space`, que randomiza os endereços de memória de alguns blocos de memória dependendo da configuração.



Quando o ASLR está configurado com o valor 2 o ASLR randomiza as posições da Stack, [VDSO](Apresenta%C3%A7%C3%A3o%20ROP.md#VDSO-%28Virtual-Dynamic-Shared-Object%29), das áreas de memória compartilhada e da região .data do binário. Geralmente esse é o valor padrão.



Quando o ASLR está configurado com o valor 1 o ASLR randomiza as posições Quando o ASLR está configurado com o valor 2 o ASLR randomiza as posições da Stack, [VDSO](Apresenta%C3%A7%C3%A3o%20ROP.md#VDSO-%28Virtual-Dynamic-Shared-Object%29) e das áreas de memória compartilhada. A região .data do binário é alocada imediatamente após o final do código executável.



Quando o ASLR está configurado com o valor 0 o ASLR está desativado.



*O ASLR com tudo não é completamente aleatório os endereços utilizados para randomizar começam a se repetir após algumas execuções do binário, sendo assim é possivel executar ataques a sistemas com ASLR criando uma lista dos endereços dos espaços de memória encontrados após algumas execuções do binário e executar uma ataque de força bruta.*



***



### Stack Canary



Canaries são valores aleatórios verificações colocadas na stack que são verificados antes do retorno de funções, esse valores serem para indicar uma possível manipulação da stack por um [BOF](Apresenta%C3%A7%C3%A3o%20ROP.md#BOF-%28Buffer-Overflow%29) o que causa a interupção da execução.



*No geral não é possível saber qual será o valor do canary antes de se executar o programa, então é necesssário um vazar o enederço do canary em tempo de execução ou executar um ataque de força bruta para burlar essa proteção.*



***



### NX (No eXecute)



É um tipo de proteção do binário que impede a execução de certas áreas da memória. Esta poteção faz com que não seja possivel de se executar [shellcodes](Apresenta%C3%A7%C3%A3o%20ROP.md#Shellcode) no programa.



*Geralmente quando o NX está habilitado são utilizados ROP Gadgets para se burlar a proteção (geralmente usando a função mprotect).*



***



### PIE (Position Independent Executable)



É um tipo de proteção que carrega as funções do programa em enedereços de memória aleatórios.



*Apesar de serem aleatórios as áreas anida tem o mesmo offset sendo assim caso seja possível encontrar o endereço de memória de um local epecífico, stack por exemplo é possível se calcular todos os demais endereços do programa.*



***



### BOF (Buffer Overflow)



Buffer Overflow é uma vunerabilidade a qual qualquer código que programa que armazene dados em um buffer de memória e não cheque seu limite corretamente está sucetível. Ataravés do buffer overflow pode se escrever dados em áreas onde isso não era suposto a acontecer, sobreescrevendo as funções do programa para executar [shellcodes](Apresenta%C3%A7%C3%A3o%20ROP.md#Shellcode) ou causar uma falha no mesmo.



#### Shellcode



É um pedaço de código inserido geralmente atarvés de um [BOF](Apresenta%C3%A7%C3%A3o%20ROP.md#BOF-%28Buffer-Overflow%29) que tem a intenção de causar a execução arbritária de código no programa (geralmente a abertura de um shell).



*No geral o shellcode só é usado quando a área onde ele é inserido para a execução não está protegida, geralmente por [NX](Apresenta%C3%A7%C3%A3o%20ROP.md#NX-%28No-eXecute%29). No caso da região estar protegida geralmente se tenta acressnetar permissão de execução a região para poder burlar o [NX](Apresenta%C3%A7%C3%A3o%20ROP.md#NX-%28No-eXecute%29) e executar o shellcode.*



#### ROP (Return Oriented Programming)



É um tipo de ataque que busca através de um [BOF](Apresenta%C3%A7%C3%A3o%20ROP.md#BOF-%28Buffer-Overflow%29) tomar controle do fluxo de execução do programa e executar código arbitráriamente.



#### ROP Gedget



São pequenos pedaços de códigos presenets no binário ou em bibliotecas associadas a ele que podem ser usados para executar códigos arbitráriamente.



***



#### Links:



- https://www.linkedin.com/pulse/elf-linux-executable-plt-got-tables-mohammad-alhyari (PLT e GOT)

- https://ctf101.org/binary-exploitation/what-is-the-got/ (PLT e GOT)

- https://www.redhat.com/en/blog/hardening-elf-binaries-using-relocation-read-only-relro (RELRO)

- https://www.networkworld.com/article/3331199/what-does-aslr-do-for-linux.html (ASLR)

- https://en.wikipedia.org/wiki/.bss (BSS)

- https://mcuoneclipse.com/2013/04/14/text-data-and-bss-code-and-data-size-explained/ (BSS)

- https://www.youtube.com/watch?v=UdMRcJwvWIY (Link dinâmico e estático, Lazy e Immediate Binding)

- https://www.youtube.com/watch?v=5FJxC59hMRY (ASLR e ROP)

- https://en.wikipedia.org/wiki/X86-64 (Endereços canônicos e não canônicos)

- https://docs.oracle.com/en/operating-systems/oracle-linux/6/security/ol_aslr_sec.html (ASLR)

- https://man7.org/linux/man-pages/man7/vdso.7.html (VDSO)

- https://ctf101.org/binary-exploitation/no-execute/ (NX)

- https://ir0nstone.gitbook.io/notes/types/stack/pie (PIE)

- https://ctf101.org/binary-exploitation/stack-canaries/ (Stack Carnary)

- https://www.youtube.com/watch?v=gxU3e7GbC-M (Basicamente todo o conteúdo condensado em 4 horas de vídeo)

- https://soaresligia.medium.com/buffer-overflow-no%C3%A7%C3%B5es-b%C3%A1sicas-26745e54668 (BOF)

- https://gitbook.ganeshicmc.com/pwning/shellcode (Shellcode)

- https://www.ired.team/offensive-security/code-injection-process-injection/binary-exploitation/rop-chaining-return-oriented-programming (ROP e exemplos)

- https://ctf101.org/binary-exploitation/return-oriented-programming/ (ROP)
