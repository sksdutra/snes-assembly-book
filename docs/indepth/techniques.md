# Técnicas
Nesse capítulo focaremos em técnicas gerais que você pode usar no ASM de SNES.

## Contagem de bytes
Cada instrução é montada em um ou mais bytes. Isso acontece como se segue: primeiro, o opcode é garantidamente montado em um byte. Em seguida, o byte é seguido por 0-3 bytes, que serve como parâmetro do opcode. Cada 2 dígitos hexadecimais é igual a 1 byte. Isso significa que a quantidade máxima de bytes que uma instrução em SNES pode usar é de quatro bytes: opcode + um parâmetro de 24 bits. Aqui está um exemplo de como seria o LDA, quando montado com diferentes modos de endereçamento:

```
LDA #$00           ; A9 00
LDA #$0011         ; A9 11 00
LDA $00            ; A5 00
LDA $0011          ; AD 11 00
LDA $001122        ; AF 22 11 00
```

Uma instrução sem um parâmetro hexadecimal tem apenas 1 byte, como `INC A` ou` TAX`. Uma instrução com um parâmetro 8-bit tem 2 bytes, como `LDA #$00`. Uma instrução com um parâmetro 16-bit tem 3 bytes, como `LDA $0000`. Uma instrução com um parâmetro 24-bit tem 4 bytes, como `LDA $000000`. Não importa se o modo de endereçamento é indexado, indireto direto ou outro. Tudo depende do comprimento do valor `$`.

## Abreviando as comparações com zero
A comparação mais rápida usa os flags do processador de forma eficaz, pois os desvios dependem, na verdade, dos flags do processador. Conforme mencionado neste tutorial anteriormente, `BEQ` efetua o desvio se o flag zero estiver habilitado,` BNE` efetua o desvio se o flag zero estiver desabilitado, `BCC` efetua o desvio se o flag carry estiver desabilitado e assim por diante.

Em geral, se o resultado de qualquer operação for zero (`$00` ou `$0000` no modo 16-bits), o flag zero é habilitado. Por exemplo, se você executar `LDA #$00`, o flag zero será habilitado. Quase todos os opcodes que modificam um endereço ou registrador afetam o flag zero.

By making use of the zero flag, it's possible to check if a certain instruction has zero as its result (including loads). This way, you can write shorthand methods to check if an address actually contains the value `$00` or `$0000`. Here's an example:

Usando o flag zero, é possível verificar se uma determinada instrução tem como resultado zero (incluindo carregamentos). Assim, você pode escrever métodos abreviados para verificar se um endereço realmente contém o valor `$00` ou` $0000`. Como no exemplo abaixo:

```
LDA $59
BEQ IsZero

LDA #$01
STA $02

IsZero:
RTS
```
Esse código checa se o endereço $7E0059 contém o valor $00. Se ele contiver, então imediatamente retorna, caso contrário, armazena o valor $01 no endereço $7E0002.

O contrário também é possível. Checar se um endereço *não* contém zero, eis um exemplo:

```
LDA $59
BNE IsNotZero

LDA #$01
STA $02

IsNotZero:
RTS
```
Esse código checa se o endereço $7E0059 *não* contém o valor $00. Se ele *não* contiver, então imediatamente retorna, caso contrário, armazena o valor $01 no endereço $7E0002.

## Looping
Os loops podem ser descritos como fluxo de código que permite a você executar código repetidamente. É muito útil quando você precisa ler uma tabela ou uma região de memória valor por valor. Um exemplo prático é a leitura de dados de nível da ROM do SNES, a fim de construir um nível jogável. Os níveis são uma longa lista de dados de objetos e sprites, portanto, esses dados podem ser lidos de forma repetitiva, até chegar ao fim dos mesmos.

Os exemplos nesta seção apenas percorrem as tabelas para copiá-las para endereços de RAM, mas é claro, os loops podem ser usados para implementar uma lógica muito mais complexa.

### Looping com verificação de um contador
É possível criar um loop do início ao fim de uma tabela. O código a seguir demonstra isso:

```
   LDX.b #$00      ; Iniciliza o contador do loop
-  LDA Table, x    ; Lê os valores na tabela
   STA $00, x      ; Armazena os valores nos endereços $7E0000-$7E0003
   INX             ; Incrementa o contador em um
   CPX.b #Table_end-Table ; Usa o tamanho da tabela como checagem
   BNE -           ; Se o contador e a checagem não forem iguais, o loop continua
   RTS

Table:   db $01,$02,$04,$08 ; Os valores são lidos nessa ordem
.end
```
Esse código inicializa um contador para o loop com o valor $00. A cada iteração, esse loop incrementa o contador em um e, então, o compara com o tamanho da tabela. Assim, o loop executa para cada byte nesta tabela.

### Looping com uma checagem negativa
Embora seja possível fazer um loop do início ao final de uma tabela, também é possível fazer um loop do final em direção ao início da tabela. Este método tem uma verificação "abreviada" para ver se o loop deve ser encerrado, fazendo uso inteligente do flag de processador "negative". Ao usar este método, seu loop irá essencialmente rodar para trás.

```
   LDX.b #Table_end-Table-1 ; Recebe o comprimento da tabela ($03). O -1 é necessário, 
-  LDA Table, x             ; senão o loop passará em 1. Esta linha usa o valor como contador
   STA $00, x               ; Armazena os valores nos endereços $7E0003 a $7E0000
   DEX                      ; Decrementa o contador em um
   BPL -                    ; Se o contador de loop não for negativo, continua
   RTS

Table:   db $01,$02,$04,$08 ; Os valores serão lidos na ordem inversa
.end
```
Como o contador de loops serve como um índice para endereçar $7E0000, assim como a tabela, ele lê e armazena os valores em ordem inversa, já que o contador começa com o valor `$03`.

O `BPL` garante que o loop será interrompido assim que o contador de loop atingir o valor` $FF`. Isso significa que o sinalizador negativo será definido, portanto, o BPL não ramificará para o início do loop. 

No entanto, esse método de loop tem uma desvantagem. Quando o contador de loop é `$81- $FF`, o loop será executado uma vez e será interrompido imediatamente, pois o BPL não se ramificará. Isso ocorre porque depois de `DEX`, o contador de loop é imediatamente negativo. Lembre-se de que os valores de `$80` a` $FF` são considerados negativos. No entanto, se seu contador de loop inicial for `$80`, ele executará primeiro o código dentro do loop, diminuirá o contador de loop em 1, *e  então* verificará se o contador de loop é negativo. Portanto, com esse loop, você pode percorrer 129 bytes de dados no máximo.

It's also possible to have the loop counter at 16-bit mode while having the accumulator at 8-bit mode. The following example demonstrates this:

Também é possível ter o contador de loop no modo de 16-bit enquanto o acumulador está no modo de 8-bit. O exemplo a seguir demonstra isso:

```
   REP #$10
   LDX.w #Table_end-Table-1 ; Recebe o comprimento da tabela ($03). Perceba o ".w" que força
-  LDA Table,x              ; o assembler a usar um valor 16-bit
   STA $00,x                ; Armazena os valores nos endereços $7E0000-$7E0003
   DEX                      ; Decrementa o contador em 1
   BPL -                    ; Se o contador de loop não for negativo, continua
   SEP #$10
   RTS

Table:   db $01,$02,$04,$08
.end
```
Nesse caso, é possível percorrer 32.769 bytes de dados no máximo.

### Loop checando o fim dos dados
Esse método basicamente mantém o loop e a iteração através dos valores, até atingir algum tipo de marcador de "fim dos dados". De modo geral, esse valor é algo que o código normalmente nunca usa como um valor real. Na maioria dos casos, é o valor `$FF` ou` $FFFF`, embora a decisão seja inteiramente do programador. Aqui está um exemplo que usa esse marcador.

Esse tipo de loop é especialmente útil quando ele precisa processar várias tabelas com a mesma lógica, mas com tamanhos de tabela variados. Um exemplo de tal implementação são os carregamentos de níveis; A lógica para analisar os níveis é sempre a mesma, mas os níveis variam em tamanho.

```
   LDX.b #$00      ; Inicializa o indexador.
-  LDA Table,x     ; Lê o valor da tabela de dados.
   CMP #$FF        ; Se fot o marcador de fim dos dados, sai do loop.
   BEQ +
   STA $00,x       ; Caso contrário, armazena o valor.
   INX             ; Incrementa o index para a tabela
   BRA -           ; Continua o loop
+  RTS

Table:   db $01,$02,$04,$FF
.end
```
No caso deste exemplo, o código percorre quatro bytes de dados, três dos quais são dados reais e um dos quais é o marcador de fim dos dados. Assim que o loop encontrar este marcador, neste caso o valor `$FF`, o loop é encerrado imediatamente.

## Bigger branch reach
No capítulo [comparações, desvios e labels](../programming/branches), é mencionado que os ramos têm uma distância limitada de -128 a 127 bytes. Quando você excede esse limite, o montador detecta isso automaticamente e lança algum tipo de erro, como o seguinte:

```
            LDA $00
            CMP #$03
            BEQ SomeLabel ; Desvia quando o endereço $7E0000 contiver o valor $03
            NOP #1000     ; 1000 vezes "NOP"
SomeLabel:  RTS
```
Esse código causará o seguinte erro:
```
file.asm:3: error: (E5037): Relative branch out of bounds. (Distance is 1000). [BEQ SomeLabel]
```
Isso significa que a distância entre o desvio e o label é de 1000 bytes, o que definitivamente excede o limite de 127 bytes. Se você necessariamente tem que pular para aquele label, você pode inverter a condicional e usar o opcode `JMP` para um alcance de salto mais longo:

```
            LDA $00
            CMP #$03
            BNE +         ; Salta o pulo quando endereõ $7E0000 não contiver o valor $03
            JMP SomeLabel ; Esse salto ocorrerá se o endereço $7E0000 contiver o valor $03

+           NOP #1000     ; 1000 vezes "NOP"
SomeLabel:  RTS
```

Dessa forma, a lógica continua a mesma, ou seja, os mil NOPs rodam quando o endereço $7E0000 não contém o valor $03. O erro fora dos limites foi resolvido. Além disso, substituir o `JMP` por` JML` permitirá que você pule *para qualquer lugar* em vez de ficar restrito ao banco atual.

## Tabelas de ponteiros
Uma tabela de ponteiros é uma tabela com uma lista de ponteiros. Dependendo do contexto, os ponteiros podem apontar para código ou dados. As tabelas de ponteiros são especialmente úteis se você precisar executar certas rotinas ou acessar certos dados para uma lista exaustiva de valores. Sem as tabelas de ponteiros, você teria que fazer uma grande quantidade de comparações manuais. Aqui está um exemplo de uma versão manual:

```
  LDA $14
  CMP #$00
  BNE +
  JMP FirstRoutine

+ CMP #$01
  BNE +
  JMP SecondRoutine

+ CMP #$02
  BNE +
  JMP ThirdRoutine

+ RTS

FirstRoutine:
  LDA #$95
  STA $15
  RTS

SecondRoutine:
  LDA #$95
  STA $15
  RTS

ThirdRoutine:
  LDA #$95
  STA $15
  RTS
```
Você pode imaginar que, com muitos valores associados a rotinas, essa lógica de comparação pode ficar enorme rapidamente. É aqui que as tabelas de ponteiros são úteis.

Isto aqui é uma tabela de ponteiros:

```
Pointers: dw Label1
          dw Label2
          dw Label3
          dw Label4
```
Como você pode ver, nada mais é do que um monte de entradas de tabela apontando para algum lugar. Você pode usar rótulos para apontar para ROM ou definições para apontar para RAM.

### Tabelas de ponteiros para código
Há algumas instruções projetadas para fazer uso de tabelas de ponteiros. São as seguintes:

|Instrução|Exemplo|Explicação|
|-|-|-|
|**JMP (*absolute address*)**|JMP ($0000)|Salta para um endereço absoluto localizado no endereço $7E0000.|
|**JMP (*absolute address*,x)**|JMP ($0000,x)|Salta para um endereço absoluto localizado no endereço $7E0000, que é indexado por `X`.|
|**JML [*absolute address*]**|JML [$0000]|Salta para um endereço longo, localizado no endereço $7E0000.|
|**JSR (*absolute address*,x)**|JSR ($0000,x)|Salta para um endereço absoluto localizado em $7E0000, que é indexado por `X`, então retorna.|
Com estes opcodes, assim como uma tabela de ponteiros, é possível executar uma subotina dependendo do valor de um determinado endereço de RAM. Aqui está um exemplo que executa uma rotina dependendo do valor do endereço da RAM $7E0014:

```
LDA $14            ; Carrega o valor em A...
ASL A              ; ...Multiplica-o por dois...
TAX                ; ...e o transfere para X
JSR (Pointers,x)   ; Executa rotinas.
RTS

Pointers: dw Label1 ; $7E0014 = $00
          dw Label2 ; $7E0014 = $01
          dw Label3 ; $7E0014 = $02
          dw Label4 ; $7E0014 = $03

Label1:   LDA #$01
          STA $09
          RTS

Label2:   LDA #$02
          STA $09
          RTS

Label3:   LDA #$03
          STA $99
          RTS

Label4:   LDA #$55
          STA $66
          RTS
```
A explicação rápida é que dependendo do valor do endereço da RAM $14, as quatro rotinas são executadas. Para o valor `$00`, a rotina em` Label1` é executada. Para o valor `$01`, a rotina em` Label2` é executada e assim por diante.

A explicação detalhada é que carregamos um valor em A e o multiplicamos por dois, porque usamos *words* para nossas tabelas de ponteiros. Portanto, precisamos indexar a cada dois bytes em vez de cada byte. Isso significa que o valor `$00` permanece como valor de indexação ` $00`, portanto, lendo o ponteiro `Label1`. O valor `$01` torna-se o valor de indexação ` $02`, lendo assim o ponteiro `Label2`. O valor `$02` torna-se o valor de indexação ` $04`, lendo assim o ponteiro `Label3`. O valor `$03` torna-se o valor de índice` $06`, lendo assim o ponteiro `Label4`. Como o JSR usa um modo de endereçamento "absoluto, indireto", os labels também são absolutos, portanto, eles só são executados no mesmo banco que esse JSR.

### Tabelas de ponteiros para dados
O mesmo conceito pode ser aplicado para dados (ou seja, tabelas). Imagine que você deseja ler os dados do nível, dependendo do número do nível. Uma tabela de ponteiros seria uma solução perfeita para isso. Aqui está um exemplo:

```
  LDA $14            ; Carrega o número do nível em A...
  ASL A              ; ...Multiplica-o por três...
  CLC
  ADC $14
  TAY                ; ...e então o transfere para Y
  LDA Pointers,y
  STA $00
  LDA Pointers+1,y
  STA $01
  LDA Pointers+2,y ; Armazena o endereço apontado na RAM
  STA $01          ; Para usar um ponteiro indireto

  REP #$10
  LDY #$0000
- LDA [$00],Y      ; Lê os dados o nível até que alcance o marcado de fim dos dados
  CMP #$FF
  BEQ Return

  INY
  BRA -
  ; Faz alguma coisa com os dados do nível
  
Return:
  SEP #$10
  RTS

Pointers: dl Level1 ; $7E0014 = $00
          dl Level2 ; $7E0014 = $01
          dl Level3 ; $7E0014 = $02
          dl Level4 ; $7E0014 = $03

Level1:   db $01,$02,$91,$86,$01,$82,$06,$FF

Level2:   db $AB,$91,$06,$78,$75,$FF

Level3:   db $D9,$B0,$A0,$21,$FF

Level4:   db $C0,$92,$84,$81,$82,$99,$FF
```
Na primeira seção, usamos o mesmo conceito de multiplicação de um valor para acessar uma tabela de ponteiros. Exceto que desta vez, multiplicamos por três, porque as tabelas de ponteiros contêm valores que são *longos*. Usamos este valor como um indexador para a tabela de ponteiros e armazenamos o ponteiro na RAM de $7E0000 a $7E0002, em little-endian. Depois disso, na segunda seção, usamos RAM $7E0000 como um ponteiro indireto e começamos a percorrer seus valores, usando `Y` como índice novamente. Continuamos o loop indefinidamente, até atingirmos um marcador de "fim dos dados", neste caso o valor `$FF`. Usamos esse método porque os níveis podem variar em comprimento. Também usamos `Y` 16-bit porque os dados de nível *podem* ter mais de 256 bytes de tamanho. Finalmente, terminamos a rotina definindo `Y` de volta para 8-bit e, em seguida, retornando.

Este exemplo também mostra como usar ponteiros 24-bit em vez de ponteiros de 16-bits. A tabela de ponteiros contém valores longos. Usamos isso em combinação com um modo de endereçamento "direto, indireto *longo*" (ou seja, os colchetes).

## Pseudo matemática 16-bit
É possível executar `ADC` e` SBC` 16-bit sem realmente alternar para o modo 16-bit. Na verdade, isso é bastante útil nos casos em que um valor 16-bit é armazenado em dois endereços separados como dois valores 8-bit. Isso é possível com a ajuda do flag carry, bem como o comportamento dos opcodes `ADC` e` SBC`.

Pseudo 16-bit math also works with `INC` and `DEC`, although you'd have to use them on the addresses instead of the A, X and Y registers. By making clever usage of the negative flag, it's possible to perform pseudo 16-bit math with this opcode also.

### ADC
Here's an example of a pseudo 16-bit `ADC`:

```
LDA #$F0
STA $00            ; Initialize address $7E0000 to value $F0 for this example
LDA #$05
STA $59            ; Initialize address $7E0059 to value $05 for this example
                   ; These would make the 16-bit value $05F0

LDA $00            ; Load the value $F0 into A
CLC                ; Clear Carry flag for addition. C = 0
ADC #$20           ; $F0+$20 = $10, C = 1
STA $00            ; $7E0000 has now the value $10

LDA $59            ; Load the value $05 into A
ADC #$00           ; Add $00 to $7E0005. BUT because C = 1, this adds $01 to A instead
STA $59            ; A is now $06, and we store it into $7E0059
                   ; These would now make the 16-bit value $0610
                   ; across two addresses
```
The carry flag is set after the first `ADC`. This means that the value has wrapped back to `$00` and increased from there. Because the carry flag is set, the second `ADC` adds $00 + carry, thus `$01`, thus increasing the second address by one.

### SBC
Here's an example of a pseudo 16-bit `SBC`:

```
LDA #$10
STA $00            ; Initialize address $7E0000 to value $10 for this example
LDA #$05
STA $59            ; Initialize address $7E0059 to value $05 for this example
                   ; These would make the 16-bit value $0510

LDA $00            ; Load the value $10 into A
SEC                ; Set Carry flag for subtraction. C = 1
SBC #$20           ; $10-$20 = $F0, C = 0
STA $00            ; $7E0000 has now the value $10

LDA $59            ; Load the value $05 into A
ADC #$00           ; Subtract $00 from $7E0005. BUT because C = 0, this subtracts $01 from A instead
STA $59            ; A is now $04, and we store it into $7E0059
                   ; These would now make the 16-bit value $04F0
                   ; across two addresses
```
The carry flag is cleared after the first `SBC`. This means that the value has wrapped back to `$FF` and decreased from there. Because the carry flag is cleared, the second `SBC` subtracts $00 + carry, thus `$01`, thus decreasing the second address by one.

### INC
Here's an example of a pseudo 16-bit `INC`:

```
   LDA #$FF
   STA $00          ; Initialize address $7E0000 to value $FF for this example
   LDA #$03
   STA $59          ; Initialize address $7E0059 to value $03 for this example
                    ; These would make the 16-bit value $$03FF

   INC $00          ; The value in $7E0000 is increased by 1, making it have the value $00
   BNE +            ; This sets the zero flag, thus the branch is not taken
   INC $59          ; Thus, the value in $7E0059 is also increased by 1
+  RTS              ; These would now make the 16-bit value $0400
                    ; across two addresses
```
By making clever usage of the zero flag, we know that the result of `INC $00` is actually the value `$00`, because that's the only time the zero flag is set. If the result is indeed the value `$00`, then the other address needs to be increased also.

### DEC
Here's an example of a pseudo 16-bit `DEC`:

```
   LDA #$00
   STA $00          ; Initialize address $7E0000 to value $00 for this example
   LDA #$03
   STA $59          ; Initialize address $7E0059 to value $03 for this example
                    ; These would make the 16-bit value $$0300

   DEC $00          ; Decrease the value in $7E0000 by 1
   LDA $00          ; If it results in $FF, then we wrapped from the value $00 to $FF
   CMP #$FF         ; Thus, we need to decrease the value in $7E0059 also
   BNE +
   DEC $59
+  RTS              ; These would now make the 16-bit value $02FF
                    ; across two addresses
As you can see, there's an extra check for the value $FF, because there's no shorthand way to check if the result of a `DEC` is exactly the value $FF. If the result indeed is the value `$FF`, then the other address needs to be decreased also.
```

## ADC and SBC on X and Y
Increasing and decreasing A by a certain amount is easy because of `ADC` and `SBC`. However, these kind of instructions do not exist for X and Y. If you want to increase or decrease X and Y by a small amount, you would have to use `INX`, `DEX`, `INY` and `DEY`. This quickly gets impractical if you have to increase or decrease X and Y by great numbers (5 or more) though. In order to do that, you can temporarily transfer X or Y to A, then perform an `ADC` or `SBC`, then transfer it back to X or Y. 

### Addition
Here's an example using `ADC`:
```
TXA                ; Transfer X to A. A = X
CLC                ; 
ADC #$42           ; Add $42 to A
TAX                ; Transfer A to X. X has now increased by $42
```
By temporarily transferring X to A and back, the `ADC` practically is used on the X register, instead.

### Subtraction
Here's an example using `SBC`:
```
TXA                ; Transfer X to A. A = X
SEC                ; 
SBC #$42           ; Subtract $42 from A
TAX                ; Transfer A to X. X has now decreased by $42
```
By temporarily transferring X to A and back, the `SBC` practically is used on the X register, instead.