# Técnicas
Nesse capítulo focaremos em técnicas gerais que você pode usar no ASM de SNES.

## Contagem de bytes
Cada instrução é montada em um ou mais bytes. Isso acontece como se segue: primeiro, o opcode é garantidamente montado em um byte. Em seguida, o byte é seguido por 0-3 bytes, que serve como parâmetro do opcode. Cada 2 dígitos hexadecimais é igual a 1 byte. Isso significa que a quantidade máxima de bytes que uma instrução em SNES pode usar é de quatro bytes: opcode + um parâmetro de 24-bit. Aqui está um exemplo de como seria o LDA, quando montado com diferentes modos de endereçamento:

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

Em geral, se o resultado de qualquer operação for zero (`$00` ou `$0000` no modo 16-bit), o flag zero é habilitado. Por exemplo, se você executar `LDA #$00`, o flag zero será habilitado. Quase todos os opcodes que modificam um endereço ou registrador afetam o flag zero.

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

## Maior alcance para os desvios
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

Este exemplo também mostra como usar ponteiros 24-bit em vez de ponteiros de 16-bit. A tabela de ponteiros contém valores longos. Usamos isso em combinação com um modo de endereçamento "direto, indireto *longo*" (ou seja, os colchetes).

## Pseudo matemática 16-bit
É possível executar `ADC` e` SBC` 16-bit sem realmente alternar para o modo 16-bit. Na verdade, isso é bastante útil nos casos em que um valor 16-bit é armazenado em dois endereços separados como dois valores 8-bit. Isso é possível com a ajuda do flag carry, bem como o comportamento dos opcodes `ADC` e` SBC`.

A pseudo matemática 16-bit também funciona com `INC` e` DEC`, embora você tenha que usá-los nos endereços ao invés dos registradores `A`, `X` e `Y`. Fazendo uso inteligente do flag negative, é possível realizar pseudo matemática 16-bit com estes opcodes também.

### ADC
Abaixo um exemplo de um pseudo matemática 16-bit com `ADC`:

```
LDA #$F0
STA $00            ; Inicializa o endereço $7E0000 como valor $F0
LDA #$05
STA $59            ; Inicializa o endereço $7E0059 como valor $05
                   ; Isso geraria o valor 16-bit $05F0

LDA $00            ; Carrega o valor $F0 em A
CLC                ; Desabilita o flag carry para adição. C = 0
ADC #$20           ; $F0+$20 = $10, C = 1
STA $00            ; $7E0000 agora tem o valor $10

LDA $59            ; Carrega o valor $05 em A
ADC #$00           ; Adiciona $00 a $7E0005. MAS como C = 1, adiciona $01 em A
STA $59            ; A agora é $06, e é armazenado em $7E0059
                   ; Assim teríamos o valor 16-bit $0610
                   ; perpassando dois endereços
```
O flag carry é definido após o primeiro `ADC`. Isso significa que o valor voltou para `$00` e aumenta a partir daí. Como o flag carry está habilitado, o segundo `ADC` adiciona $00 + carry, portanto,` $01`, aumentando assim o segundo endereço em um.

O flag carry é habilitado após o primeiro `ADC`. Isso significa que o valor voltou para `$00` e aumentou a partir daí. Como o flag carry está habilitado, o segundo `ADC` adiciona $00 + carry, portanto,` $01`, aumentando assim o segundo endereço em um.

### SBC
Abaixo um exemplo de um pseudo matemática 16-bit com `SBC`:

```
LDA #$10
STA $00            ; Inicializa o endereço $7E0000 com o valor $10
LDA #$05
STA $59            ; Inicializa o endereço $7E0059 como valor $05
                   ; Isso geraria o valor 16-bit $05F0

LDA $00            ; Carrega o valor $F0 em A
SEC                ; Habilita o flag carry para subtração. C = 1
SBC #$20           ; $10-$20 = $F0, C = 0
STA $00            ; $7E0000 agora tem o valor $10

LDA $59            ; Carrega o valor $05 em A
ADC #$00           ; Subtrai $00 de $7E0005. MAS como C = 0, subtrai $01 de A
STA $59            ; A agora é $04, e é armazenado em $7E0059
                   ; Assim teríamos o valor 16-bit $04F0
                   ; perpassando dois endereços
```
O flag carry é desabilitado após o primeiro `SBC`. Isso significa que o valor voltou para `$FF` e diminuiu a partir daí. Como o flag carry está desabilitado, o segundo `SBC` subtrai $ 00 + carry, portanto` $01`, diminuindo assim o segundo endereço em um.

### INC
Abaixo um exemplo de um pseudo matemática 16-bit com `INC`:

```
   LDA #$FF
   STA $00          ; Inicializa o endereço $7E0000 com o valor $FF
   LDA #$03
   STA $59          ; Inicializa o endereço $7E0059 com o valor $03
                    ; Isso irá compor o valor 16-bit $$03FF

   INC $00          ; O valor em $7E0000 é incrementado em 1, gerando o valor $00
   BNE +            ; Isso habilita o flag zero, assim o desvio não é feito
   INC $59          ; Aqui, o valor em $7E0059 também é incrementado em 1
+  RTS              ; Isso geraria o valor 16-bit $0400
                    ; perpassando dois endereços
```
Fazendo uso inteligente do flag zero, sabemos que o resultado de `INC $00` é na verdade o valor` $00`, porque é a única vez que o flag zero é definido. Se o resultado for realmente o valor `$00`, então o outro endereço também precisa ser incrementado.

### DEC
Abaixo um exemplo de um pseudo matemática 16-bit com `DEC`:

```
   LDA #$00
   STA $00          ; Inicializa o endereço $7E0000 com o valor $00
   LDA #$03
   STA $59          ; Inicializa o endereço $7E0059 com o valor $03
                    ; Isso irá compor o valor 16-bit $$0300

   DEC $00          ; Decrementa o valor em $7E0000 em 1
   LDA $00          ; Se resultar em $FF, então passamos do valor $00 para $FF
   CMP #$FF         ; Então, precisamos decrementar o valor em $7E0059 também
   BNE +
   DEC $59
+  RTS              ; Isso geraria o valor 16-bit $02FF
                    ; perpassando 2 endereços
```
Como você pode ver, há uma verificação extra para o valor $FF, porque não há uma forma abreviada de verificar se o resultado de um `DEC` é exatamente o valor $FF. Se o resultado realmente for o valor `$FF`, então o outro endereço também precisa ser decrementado.

## ADC e SBC em X e Y
Increasing and decreasing A by a certain amount is easy because of `ADC` and `SBC`. However, these kind of instructions do not exist for X and Y. If you want to increase or decrease X and Y by a small amount, you would have to use `INX`, `DEX`, `INY` and `DEY`. This quickly gets impractical if you have to increase or decrease X and Y by great numbers (5 or more) though. In order to do that, you can temporarily transfer X or Y to A, then perform an `ADC` or `SBC`, then transfer it back to X or Y. 

### Adição
Abaixo um exemplo usando `ADC`:
```
TXA                ; Transfer X to A. A = X
CLC                ; 
ADC #$42           ; Add $42 to A
TAX                ; Transfer A to X. X has now increased by $42
```
By temporarily transferring X to A and back, the `ADC` practically is used on the X register, instead.

### Subtração
Abaixo um exemplo usando `SBC`:
```
TXA                ; Transfer X to A. A = X
SEC                ; 
SBC #$42           ; Subtract $42 from A
TAX                ; Transfer A to X. X has now decreased by $42
```
By temporarily transferring X to A and back, the `SBC` practically is used on the X register, instead.