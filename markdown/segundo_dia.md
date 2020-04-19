*Leia ouvindo*

<iframe src="https://open.spotify.com/embed/track/4PevTpnIWy2D1wIQZY3MmX" width="300" height="80" frameborder="0" allowtransparency="true" allow="encrypted-media"></iframe>

### x86 refresher pt. 2

Se você não viu o vídeo que redirecionava pra essa página, eu não te culpo, falhei várias vezes em colocar o vídeo em autoplay... mas, enfim, se quiser ver é só voltar e clicar na tela. Esse vídeo foi feito pela NISEnet: https://www.youtube.com/watch?v=Knd-U-avG0c&list=WL&index=16&t=157s. Obrigada NISEnet por sempre me lembrar do mundo mágico que é a computação╰(◡‿◡✿╰)

Bom, no post passado eu falei que estávamos passando por um momento de "refrescar" a mente enquanto o curso da Offsec não começava, por isso, bem vindo a parte dois do nosso momento refreshing! (ﾉ◕ヮ◕)ﾉ*:･ﾟ✧

![pool](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/pool.gif)

Nós estávamos vendo esse código aqui oh:

```assembly
;hello_world.asm

global _start
section .text
_start:
		;print hello world
		mov eax, 0x4
		mov ebx, 0x1
		mov ecx, message
		mov edx, mlen
		int 0x80
		
		;exit
		mov eax, 0x1
		mov ebx, 0x5
		int 0x80
		
section .data
		message: db "Hello World!"
		mlen 		 equ $-message
```
Se tudo ocorreu bem, você deve estar vendo um "Hello World!" depois de rodar ./helloworld (ou qualquer outro nome que você colocou para o seu código). 

Legal...

...mas que que ta rolando?

Para entender melhor, antes, é bacana conhecer alguns outros conceitos. Então bora pro nosso #tbt hehe

Se você leu o post passado, você já entendeu o processo de digestão macro do código até chegar a nível de máquina (famoso binário), podemos chamar de "Assembly x86 toolchain", entretanto, é importante compreender o *modelo de alocação do código em memória* para ajudar-nos a cada vez mais contruir a nossa capacidade de abstração quando analisando assembly. Foi em um MOOC israelense chamado "From NAND to Tetris" que eu ouvi "*Computer science is a thousand layers of abstraction*" do professor **Shimon Schocken**, depois desse dia eu nunca mais estudei computação da mesma forma... 

**Bullshitting sobre minha vida** *pode pular se quiser* 

Eu passei por uma transição bem engraçada desde o dia que comecei em computação pra hoje. Até os 22 anos eu não imaginava que o computador era mais que uma carcaça mágica para responder emails, pesquisar no google e usar o facebook, nessa época eu estava mais preocupada em entender o processo de autofagia celular e me orgulhava por conseguir fechar uma tarefa usando o task manager... rs ...até que me apresentaram o Arduino e, então, eu comecei um processo de desconstrução da tecnologia que seguiu mais ou menos nessa linha:

As coisas auto program-se --> Existe uma linguagem de programação --> Existe mais de uma linguagem de programação --> Eu posso criar uma linguagem de programação --> Eu posso criar um interpretador/compilador para essa linguagem --> Eu posso criar o hardware para embedar o compilador --> Eu não preciso de código para realizar uma ação no hardware --> Tudo não passa de ter ou não energia --> Eu posso criar uma nova arquitetura e modelos de computação se eu quiser.

![mindblown](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/mindblown.gif)

...Sim, quando eu comecei, eu achava que só existia uma linguagem de programação XD e, sim, eu tinha 22 anos, shame on me XD

A partir daí eu aprendi a importância de entender cada passinho do que estava estudando e de tentar abstrair isso na minha cabeça. Não foi rápido, eu estou a mais ou menos um ano voltando sempre nos mesmos conceitos, fazendo 3242346 desenhos diferentes e lendo 453456 artigos pra cada dia eu descobrir que sei menos. Masoquista? Talvez haha mas eu também aprendi a amar essa jornada!

​      ![cat_inlove](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/cat_inlove.gif)

**fim do bullshitting**



### Virtual Memory Model

A primeira coisa que precisamos abstrair é o modelo de memória virtual. Em poucas palavras, esse modelo divide de forma lógica a memória em *Kernel Space* e *User Space*. 

![virtual_memory](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/virtual_memory.png)

O kernel é executado em *Kernel Space*. Esta parte da memória não pode ser acessada diretamente por processos de usuários normais, mas o kernel pode acessar todas as partes da memória. Para acessar alguma parte do kernel, os processos de usuário precisam usar chamadas predefinidas do sistema, como `open`, `read`, `write`, etc. As funções da biblioteca C, como `printf`, chamam a syscall `write` para imprimir na tela, por exemplo. 

As syscalls agem como uma interface entre os processos do usuário e os processos do kernel. Os direitos de acesso são colocados no *Kernel Space* para impedir que os usuários mexam com o kernel e ferrem com o sistema sem querer :)

Portanto, quando ocorre uma chamada de syscall (redundante, eu sei rs), uma interrupção do software é enviada ao kernel. A CPU pode entregar o controle temporariamente para a rotina de tratamento de interrupção associada. O processo do kernel que foi interrompido pela interrupção é retomado depois que a rotina de tratamento da interrupção termina o que estiver fazendo.

*Obrigada @b0rk <3 https://twitter.com/b0rk/status/804200666226900992.*

Going further, em um computador 32-bit, o endereço de memória possui 32 bits e normalmente é representado em hexadecimal, variando de 0x00000000 a 0xffffffff (4GB), essa parte sempre mexe com a nossa cabeça, então, assim, porque 4gb? 2^32 ~= 4gb que é o número total de possibilidades de 0 ou 1 em 32 espaços. Significa que se pegássemos 32 espaços:

-------------------------------- e fôssemos brincando de trocar 1 ou 0 pra cada espaço:

​																						00000000000000000000000000000000

​																						00000000000000000000000000000001... teríamos 4294967296 resultados, esses "resultados" seriam os nossos endereços de memória virtual, deu pra pegar a referência? 

00000000000000000000000000000000 em binário é 0x00000000 em hex, porque cada hex tem 4 bits... ![what](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/what.gif)

Se você nunca parou para ficar abstraindo essa parte, ela pode ser bem confusa mesmo... mas uma coisa que você sempre deve ter em mente é que, no fim, tudo é binário. TUDO. Tirando o hardware em si rs, mas quando falamos de código rodando em hardware, com exceção de alguns hardwares rs, esse código vai ter que virar uma maçaroca de binário. Daí podemos fazer a análise inversa, assim:

Legal, você está me dizendo que o endereço lógico é representado em hexadecimal, como eu posso saber quantos bits um endereço tem?

1. Um bit só pode ser 1 ou 0

2. Hexadecimal varia de 0 a F

3. Como é 0 e F em binário? R: 0 e 1111

4. Hmmmmmm, tem como eu traduzir F pra binário com menos de 4 "espaços"? R: Nope.

5. Eureka, hexa tem que ter 4 espaços para poder representar todas as suas formas. Para cada "espaço" eu só posso ter 1 ou 0, ou seja, espaço == bit, logo...

   ​	 **Hexa tem 4 bits.** ![genius](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/genius.gif)

   ....Tá, mas calma, vamos continuar:

   1. Um endereço tem 8 espaços de "hexas" (oxffffffff), cada hexa tem 4 bits, um endereço tem 8x4 bits = 32 bits :o 

   2. E o que eu que posso fazer com um endereço virtual de memória? R: Você pode colocar os seus dados!! (como na imagem acima)

   3. Poxan, mas eu só posso colocar 1 byte de dados em cada endereço? R: sim ;.;

   4. Ok, então quantos bytes de dados eu posso colocar em um computador com 4294967296 possibilidades de endereços?

      ​	Se cada endereço ---- 1 byte

      4294967296 endereços ---- 4294967296 bytes

      4294967296 bytes ----  ~4 GB

      WHAAAAT? então você está dizendo que eu só posso ter 4 GB de memória em um computador de 32-bit? Mas quando eu comprei o moço da loja disse que eu tinha 1 TB de memória pra poder por fotos, ele me enganou?

      Não não, a ideia aqui é que você não vai poder tunar o seu pc de 32-bit com 16 GB de RAM porque ele não vai ter a capacidade de indexar tudo. Ele está referindo-se a memória não-volátil como a dos hard/solid-state drives e nós estamos falando de uma memória volátil, com acesso aleatório, chamada "RAM" *Random Access Memory*, também conhecida como memória principal.

      *Disclaimer* existem esquemas de endereçamento de memória que permitem que arquiteturas de 𝑘-bits (onde 𝑘 pode ser 32 ou menos) acessem mais de 2 𝑘 locais (onde um local não precisa ser restrito a bytes). Dá uma olhada no PAE(**Physical Address Extensions**,) da Intel ou lê a resposta desse cara aqui: https://www.quora.com/What-are-Intel-physical-address-extensions

      *socorro* *pq eu fui estudar esse negocio* *achei que computação era uma ciência exata, mas toda hora muda os negocio tudo*

      ​          ![crazy](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/crazy.gif)

      **relaxa, da uma respirada, vai passar rs**

      Bora continuar, dá uma olhada nesse esquema:

![memo_map](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/memo_map.png)

- **Stack**

  O stack está localizado logo abaixo do kernel do SO e uma particularidade é que ele "cresce pra baixo", ou seja ele vai dos endereços mais altos para os mais baixos (pode crescer na direção oposta em algumas outras arquiteturas). Ele usa de uma estrutura de dados chamada LIFO (Last In, First Out). Essa pilha (a.k.a stack) é um tipo de dado abstrato que serve como uma coleção de elementos, com duas "operações" principais:

  *push* que adiciona um elemento à coleção; e
  *pop* que remove o elemento mais recente. 

  ![LIFO](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/LIFO.gif)

  Esse segmento é dedicado ao armazenamento de todos os dados necessários para a chamada de uma função. Chamar uma função é o mesmo que empurrar *pushing* a execução da função chamada para o topo da pilha e, quando essa função é concluída, os resultados são retornados, retirando ou *popping* a função da pilha. O conjunto de dados enviado para a chamada de função é chamado de *stack frame* e contêm os seguintes dados:

  - os argumentos passados para a rotina de execução;
  - o endereço de retorno para o chamador da rotina;
  - espaço para as variáveis locais da rotina;

  É assim que as funções recursivas são implementadas em C, cada vez que uma função recursiva se chama, um novo quadro de pilha é alocado no topo da pilha e, assim, o conjunto de variáveis em uma chamada é completamente independente do de outra chamada de função... Depois vamos olhar o stack em mais detalhes :)

- **Heap + Shared Libs + Mappings**

  Esse é o segmento em que a alocação dinâmica de memória geralmente ocorre. A área de heap começa no final do segmento BSS e "cresce para cima", ou seja, para endereços de memória mais altos. É gerenciado pelo `malloc`/`new`, `free`/`delete`, que pode usar as syscalls `brk` e `sbrk` para ajustar seu tamanho.
  Essa área é compartilhada por todas as bibliotecas compartilhadas e módulos carregados dinamicamente em um processo. Mas nem liga muito pra isso por enquanto... :p

- **BSS**

  O segmento de "dados não inicializado", geralmente chamado de segmento BSS (por conta do antigo assembler *Block Started by Symbol*) contém variáveis que não são inicializadas (estáticas) ou que foram inicializadas com o valor "0". Por exemplo, uma variável declarada como `static int i;` seria alocada para o segmento BSS.

- **Data**

  Agora sim, aqui teremos as variáveis estáticas e globais que foram declaradas em um programa. Esse segmento pode ter ambos read-only e read-write dependendo do que for pedido. Um `const char* string = “hello world”` seria tratado como **read-only**, enquanto que `int x = 1` (declarada como global) seria tratado como **read-write**.

- **Text**

  O segmento Text guarda as instruções de máquina que devem ser executadas do programa e é **read-only/execute**. Normalmente ele é colocado "embaixo" do Stack/Heap para evitar overflows hehe Uma coisa legal de saber é que esse segmento é compartilhável de modo que apenas uma única cópia precisa estar na memória para programas que são executados com frequência, como editores de texto, o compilador C, shells, etc.

Se você chegou até aqui, high five! ![h5](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/h5.gif)

Agora, vamos dar uma olhada de novo no nosso hello_world.asm laaa em cima de novo, você já consegue ver com outros olhos? 

![segmentos](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/segmentos.png)

Acho que já é muita informação por hoje, né?

![snail_by](/Users/apercario/Documents/Courses/Offsec/Notes/nutc4k3.github.io/imgs/snail_by.gif)

*Qualquer dúvida ou correção é só me chamar de boas no twitter [@nutcake7](https://twitter.com/nutcake7)*