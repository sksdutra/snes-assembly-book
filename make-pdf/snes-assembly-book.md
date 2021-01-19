# O Livro de Assembly para SNES

# Sobre a Tradução

A tradução e revisão deste documento é um esforço coletivo de membros da cena de ROM hacking brasileira, são eles: Gambas, Israel, Joapeer e Ondinha, com o único intuito de prover documentação de qualidade e em português a todos que tenham interesse no SNES/65c816 no Brasil. Esperamos que seja útil em seu aprendizado!

# Sumário

* [Introdução](#Introdução)
* [Iniciando](#Iniciando)
* [Contribuindo](#Contribuindo)

## O Básico

* [Hexadecimal](#Hexadecimal)
* [Binário](#Binário)
* [Memória](#A-Memória-do-SNES)
* [Registradores](the-basics/registers.md)
* [Modos de endereçamento](the-basics/endereçamento.md)
* [Little-endian](the-basics/endian.md)
* [Glossário](the-basics/glossary.md)

## O Básico(novamente)

* [Carregando e armazenando](programming/loading-and-storing.md)
* [Endereços mais curtos](programming/shorter-addresses.md)
* [Modo 8 e 16-bit](programming/816.md)
* [Comparando, branches, labels](programming/branches.md)
* [Salto para sub-rotinas](programming/subroutine.md)

## Coleção de Valores

* [Tabelas e indexação](coleções/indexação.md)
* [A pilha](coleções/pilha.md)
* [Copiando dados](coleções/movimentos.md)

## Flags e Registradores do Processador

* [Flags do processador](processor/flags.md)
* [Alterando as Flags do processador](processor/repsep.md)
* [Transferências](processor/transfer.md)
* [Registrador Stack pointer](processor/stackpointer.md)

## Matemática e Lógica

* [Operações aritméticas](math/arithmetic.md)
* [Operações de deslocamento de bits](math/shift.md)
* [Operações bit a bit](math/logic.md)
* [Hardware math](math/math.md)

## Aprofundando-se

* [Modos de endereçamento revistados](indepth/addressing.md)
* [Opcodes diversos](indepth/misc.md)
* [Ciclos da máquina](indepth/cycles.md)
* [Vetores de hardware](indepth/vector.md)
* [Técnicas](indepth/techniques.md)
* [Sintaxe do assembler comum](indepth/syntax.md)
* [Precauções de programação](indepth/cautions.md)

# Introdução

Este tutorial é uma versão online do meu tutorial de assembly 65c816 que está hospedado em [SMW Central](https://www.smwcentral.net/). Originalmente, eu escrevi este tutorial para ensinar à comunidade SMW Central a linguagem assembly 65c816 em inglês simples. Hoje em dia, ele é lido por várias pessoas na cena de ROM hacking em geral. Portanto, decidi abrir o código deste tutorial no GitHub, para que as pessoas possam fazer melhorias ou traduções.

Embora eu seja um membro do SMW Central, este tutorial não está associado ao Super Mario World, portanto, este tutorial não foi feito para esse jogo. Em vez disso, este tutorial pode ser aplicado em todo o contexto SNES.

## A Linguagem

O assembly 65c816 é a linguagem usada pelo chip \(SNES\) Ricoh 5A22 do Super Nintendo Entertainment System. Dividindo as diferentes partes do acrônimo 65c816: 816 significa que o processador pode assumir o modo de 8-bits ou no modo de 16-bits. O c significa CMOS, 65 significa que este processador pertence à família de CPU's 65xx. O processador devia ser bastante revolucionário para a época. Este tutorial explica mnemônicos/instruções \(opcodes\) e como usá-los corretamente. Este tutorial não se concentra em tópicos específicos do SNES, como registradores do hardware.

Com ASM 65c816 você pode codificar coisas para os jogos de SNES \(como recursos personalizados para Super Mario World\). ASM é uma linguagem de programação de 2ª geração, de baixo nível em comparação com C\#, por exemplo. É um código de máquina legível, que eventualmente é traduzido em código de máquina hexadecimal. Todos os opcodes consistem em 3 letras, acompanhado de vários parâmetros.

## Agradecimentos Especiais

Meus agradecimentos especiais a essas pessoas que revisaram o tutorial original no SMW Central: **[spigmike](https://www.smwcentral.net/?p=profile&id=132), [Roy](https://www.smwcentral.net/?p=profile&id=845), [smkdan](https://www.smwcentral.net/?p=profile&id=411), [S.N.N](https://www.smwcentral.net/?p=profile&id=23), [andy\_k\_250](https://www.smwcentral.net/?p=profile&id=67), [Domiok](https://www.smwcentral.net/?p=profile&id=7211), [reghrhre](https://www.smwcentral.net/?p=profile&id=4176), [ChaoticFox](https://www.smwcentral.net/?p=profile&id=3462), [Tails\_155](https://www.smwcentral.net/?p=profile&id=6151), [GreenHammerBro](https://www.smwcentral.net/?p=profile&id=18802), [Vitor Vilela](https://www.smwcentral.net/?p=profile&id=8251)**

E também agradeço muito aos [contribuintes](https://github.com/Ersanio/snes-assembly-book/graphs/contributors) deste repositório!

# Iniciando

## IDE

Não há IDEs dedicados para assembly 65c816. Você pode usar qualquer editor de texto ASCII, como o Notepad ou VS Code. No entanto, algumas pessoas criaram vários plugins para editores de código existentes para adicionar recursos extras, como destaque de sintaxe:

* Plugin "[65816 Assembly](https://marketplace.visualstudio.com/items?itemName=joshneta.65816-assembly)" de Josh Neta para o VS Code;
* Plugin "[65816 SNES Assembly Language Server](https://marketplace.visualstudio.com/items?itemName=vicerust.snes-asm)" de Vice para para o VS Code;
* Plugin '"[65xx Assembly Language Support](https://atom.io/packages/language-65asm)" de MatthewCallis para o Atom.

Os arquivos escritos em código assembly geralmente são salvos com a extensão ".asm".

## Assemblers

Este tutorial usa a sintaxe que é usada por um assembler chamado "Asar", originalmente escrito por Alcaro, agora mantido por vários membros da comunidade SMW Central. Este assembler está hospedado no SMW Central e pode ser baixado [aqui](https://www.smwcentral.net/?p=section&a=details&id=19043). O repositório GitHub do Asar pode ser encontrado [aqui](https://github.com/RPGHacker/asar).

# Contribuindo
Este capítulo é destinado aos que contribuem com este tutorial. Ao escrever,  há alguns padrões que devem ser seguidos para maximizar a sua consistência ao longo do tutorial.

## Estilo
Essas diretrizes se referem ao estilo do documento.

### Tabelas
- Sentenças e frases dentro das células da tabela geralmente não devem terminar com um ponto final;
- As tabelas que apresentam os opcodes devem ter os opcodes em **negrito** e ter pelo menos as três colunas, conforme mostrado no exemplo a seguir:

| Opcode  | Nome completo         | Explicação            |
| ------- | --------------------- | --------------------- |
| **LDA** | Carrega no acumulador | Carrega um valor em A |

## Terminologia
| Regra                                                        | Exemplo                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Ao referir-se ao processador `Ricoh 5A22`, use` o SNES` em vez disso | `O SNES` é capaz de entrar no modo de 8 ou 16 bits           |
| Ao referir-se a uma área específica na memória SNES, sempre prefixe `endereço` a ela, de preferência com a área de memória (ou seja,`RAM`) | \(O\) `endereço da RAM` $7E0000 contém [...]                 |
| Ao referir-se aos registradores A, X e Y, basta usar `A`,`X` ou `Y` | Agora `A` está no modo de 8 bits. `X` é usado para indexar endereços |
| Ao referir-se a valores, sempre acrescente um `valor` antes  | A contém o `valor` $00                                       |

## Códigos de exemplo
- O código deve ser indentado quando houver rótulos, sub-rótulos ou rótulos mais ou menos na mesma linha que uma instrução. O recuo do texto indentado deve ser igual ao comprimento do rótulo mais longo mencionado anteriormente no bloco de código, incluindo os dois pontos ("`:`"), mais dois espaços adicionais;
- O código deve usar espaços em branco para indentação, sem tabulação;
- Os opcodes são escritos inteiramente em maiúsculas (ex: `LDA`);
- Os rótulos são escritas em PascalCase (ex: `Label1:`);
- As sub-rótulos são escritos em minúsculas, sublinhados e o nome deve se adequar semanticamente ao rótulo pai, sem redundância (ex: `.return`).
- As definições são escritas em PascalCase (ex: `!SomeDefine`);
- Dados diretos (`db`, `dw`, `dl`, `dd`) e especificadores de comprimento de opcode (`.b`,`.w`, `.l`) são escritos inteiramente em minúsculas;
- Os indicadores de comentários (`;`) devem começar na coluna 20 e preenchidos à esquerda por espaços em branco, não por tabulações;
    - Se não houver espaço para um comentário na coluna 20, ele começará na mesma linha;
- Os indicadores de comentários devem ser seguidos por um espaço em branco, antes do próprio comentário;
- Haverá uma nova linha extra após os opcodes `RTS`, `RTL`, `RTI`, `JMP`,  `JML`, `BRA`, `BRL`;
- Estas são orientações que devem ser seguidas o mais estritamente possível, mas pode haver casos excepcionais.

Exemplo:
```
SomeLabel:         ; Este label está em sua própria linha
LDA.b #$42
STA $00            ; Isto é um comentário
RTS

.table:
db $01,$02,$03,$04 ; Outro comentário

.second_table:
db $01,$02,$03,$04,$01,$02,$03,$04 ; Um comentário excepcional

TestLabel: LDA #$02 ; Este rótulo está na mesma linha que uma instrução
           STA $01
           BNE +
           NOP
+          RTS      ; O código está indentado de acordo com "TestLabel"
```

# O Básico

# Hexadecimal

Para programar em ASM 65c816 , você precisará entender o básico de hexadecimal. Hexadecimal, também conhecido como "hex", é um sistema numérico muito parecido com o decimal, que é o sistema de contagem que as pessoas usam diariamente. No hexadecimal, existem 6 dígitos adicionais para cada casa numérica, que são representados pelas letras A, B, C, D, E e F, conforme a tabela abaixo.

| Decimal | Hexadecimal |
| :------ | :---------- |
| 0       | 0           |
| 1       | 1           |
| 2       | 2           |
| 3       | 3           |
| 4       | 4           |
| 5       | 5           |
| 6       | 6           |
| 7       | 7           |
| 8       | 8           |
| 9       | 9           |
| 10      | A           |
| 11      | B           |
| 12      | C           |
| 13      | D           |
| 14      | E           |
| 15      | F           |
| 16      | 10          |
| 17      | 11          |
| ...     | ...         |
| 255     | FF          |

Há várias maneiras de escrever números hexadecimais para que os leitores não possam confundi-los com números decimais. São os seguintes:

* Prefixo "0x" \(0x42\)
* Prefixo "$"  \($42\)
* Sufixo  "H"  \(42H\)

Por convenção, neste tutorial usaremos o "$" para prefixar os números hexadecimais.

Em assembly, um número hexadecimal com dois dígitos é chamado de “byte”. Isso significa que os valores entre $00 a $FF são considerados como um byte.

## Valores com e sem sinal

No mundo real, os números podem ser positivos ou negativos. Em assembly, dependendo do código, os valores podem ser tratados como "com sinal" ou "sem sinal". Isso significa que os valores com sinal também podem ser negativos: Os valores de $80 para cima são considerados números negativos em decimal, começando em -128 e diminuindo conforme o número hexadecimal vai aumentando, como você pode ver na tabela abaixo.

| Decimal | Hexadecimal |
| :------ | :---------- |
| 126     | $7E         |
| 127     | $7F         |
| -128    | $80         |
| -127    | $81         |
| ...     | ...         |
| -1      | $FF         |

A presença de números negativos depende da programação do jogo. Por exemplo, um jogador pode ter velocidade positiva e negativa \(resultando em ir para frente ou para trás\), mas um jogador não pode ter vidas ou pontos extras negativos \(até porque isso não faz sentido\). Não há necessidade em dizer que o valor -0 não existe.

## Valores hexadecimais de quatro dígitos

Os números hexadecimais podem passar além dos dois dígitos, como pode ser visto abaixo.

| Decimal | Hexadecimal |
| :------ | :---------- |
| 254     | $FE         |
| 255     | $FF         |
| 256     | $0100       |
| 257     | $0101       |
| ...     | ...         |
| 65535   | $FFFF       |

O formato deste número hexadecimal é: $HHLL.

* HH é o "high byte" ou byte mais significativo;
* LL é o "low byte"  ou byte menos significativo.

# Binário

Outro sistema numérico importante é o “binário”. O binário tem apenas valores de dois dígitos para cada casa numérica: 0 e 1. Um dígito binário também é chamado de "bit". Na sintaxe do assembly, os bits são prefixados por "%".

Um *byte* é composto por oito “bits”. Como um dígito binário tem dois valores possíveis e um *byte* tem 8 *bits*, isso significa que há 2⁸(256) possibilidades em apenas um *byte*.

Por exemplo, um *byte* pode conter os seguintes *bits*: `1001 0110` ou `1001 0101`. O primeiro *bit* da esquerda é chamado de “bit 7” e o *bit* final é chamado de “bit 0”. Eles **não** são chamados de *bits* 0-7, nem *bits* 8-1. A seguir temos uma visão geral dos *bits*:

```
Bit 7654 3210

    1001 0110
    1001 0101
    .... ....
```

A tabela abaixo mostra um modo relativamente fácil de memorizar binários.

| Binário      | Hexadecimal |
| ------------ | ----------- |
| `%0000 0001` | `$01`       |
| `%0000 0010` | `$02`       |
| `%0000 0100` | `$04`       |
| `%0000 1000` | `$08`       |
| `%0001 0000` | `$10`       |
| `%0010 0000` | `$20`       |
| `%0100 0000` | `$40`       |
| `%1000 0000` | `$80`       |

Observe que há um espaço entre 4 *bits* para facilitar a leitura, embora os assemblers geralmente não aceitem esse formato. Os grupos de 4 *bits* são chamados de "nibbles" e, para os propósitos deste capítulo, eles existem para tornar o binário mais fáceis de serem lidos. Um *nibble* corresponde a um dígito em hexadecimal.

O SNES é capaz de trabalhar com números de 8 *bits* e 16 *bits*. Enquanto os números de 8 *bits* são chamados de *byte*, os números de 16 *bits* são chamados de "word". Eles tem a seguinte aparência quando convertidos para binários de 16 *bits*: `10000101 11010101` (que é iqual a `$85D5` em hexadecimal\). No caso de números de 16 *bits*, o *bit* mais à esquerda é chamado de "bit 15", enquanto o bit mais à direita é chamado de "bit 0":

```text
    1111 11             (leia de cima para baixo)
Bit 5432 1098 7654 3210
    1000 0101 1101 0101
    0000 0000 1001 0110
    .... .... .... ....
```

## Sinalizadores

O binário é imensamente útil quando você está atribuindo a um valor hexadecimal várias finalidades, como uma chave que liga e desliga determinados recursos. Esses *bits* são chamados de "Sinalizadores" e geralmente são usados para economizar espaço na memória dos jogos.

Por exemplo, você pode dividir um *byte* em 8 *bits*, com cada *bit possuir um significado diferente. O *bit* 7 pode indicar que um nível tem chuva ou não. O *bit* 6 pode indicar que um layout de nível é horizontal ou vertical. O *bit* 5 pode indicar que a configuração do nível é durante o dia ou noite, etc. Dessa forma, você pode compactar as informações em um único *byte*. Ficaria assim em binário:

```text
10100000
││└───── Indica que "Está de dia"
│└────── Indica que "Está no nível horizontal"
└─────── Indica que "Está chovendo"
```

Finalmente, aqui está uma visão geral de como contar em decimal, hexadecimal e binário:

| Decimal | Hexadecimal | Binário      |
| ------- | ----------- | ------------ |
| `00`    | `$00`       | `%0000 0000` |
| `01`    | `$01`       | `%0000 0001` |
| `02`    | `$02`       | `%0000 0010` |
| `03`    | `$03`       | `%0000 0011` |
| `04`    | `$04`       | `%0000 0100` |
| `05`    | `$05`       | `%0000 0101` |
| `06`    | `$06`       | `%0000 0110` |
| `07`    | `$07`       | `%0000 0111` |
| `08`    | `$08`       | `%0000 1000` |
| `09`    | `$09`       | `%0000 1001` |
| `10`    | `$0A`       | `%0000 1010` |
| `11`    | `$0B`       | `%0000 1011` |
| `12`    | `$0C`       | `%0000 1100` |
| `13`    | `$0D`       | `%0000 1101` |
| `14`    | `$0E`       | `%0000 1110` |
| `15`    | `$0F`       | `%0000 1111` |
| `16`    | `$10`       | `%0001 0000` |
| `17`    | `$11`       | `%0001 0001` |
| ...     | ...         | ...          |
| `254`   | `$FE`       | `%1111 1110` |
| `255`   | `$FF`       | `%1111 1111` |

## Notação

Às vezes, os *bits* podem ser escritos de forma inconsistente, como `11` ou` 110 0000`. Isso torna o número binário mais difícil de ler, porque a convenção geral é escrever *bits* em grupos de oito. Para lê-los, você precisará adicionar zeros antes dos dígitos até que haja 8 *bits* ou 16 *bits* no total.

Em 8 *bits*:

* `11` torna-se `00000011`
* `1100000` torna-se `01100000`

Em 16 *bits*:

* `11` torna-se `00000000 00000011`
* `1100000` torna-se `00000000 01100000`

Você pode converter entre decimal, hexadecimal e binário, usando o modo de "programação" da calculadora do seu sistema operacional. Existem também muitas calculadoras online que podem fazer isso. A sintaxe do assembly também aceita números decimais, portanto, geralmente não é necessário converter entre decimal e hexadecimal.

# A Memória do SNES

Escrever em assembly envolve escrever um monte de instruções em que você carrega um "valor" e o armazena em um "endereço" para obter o efeito desejado, como alterar o *powerup* do jogador por exemplo. Ao gravar dados em assembly, você trabalhará com a memória do SNES na maior parte do tempo.

A memória do SNES é basicamente uma região de *bytes*, e cada *byte* está localizado em um "endereço". Pense nisso como um tabuleiro de xadrez:

![Tabuleiro de Xadrez](/home/sandro/Documentos/snes-assembly-book/make-pdf/.gitbook/assets/chessboard.png)

Você pode ver que para se referir a uma determinada casa, a imagem faz uso de nomes de colunas e casas. Na imagem acima o "endereço" da rainha \(o "valor" \) seria o endereço D8, por exemplo. Além disso, uma única casa não pode conter duas unidades. Este mesmo conceito se aplica à memória do SNES.

A memória do SNES é mapeada do endereço $000000 a $FFFFFF, embora apenas os endereços de $00000 a $7FFFFF sejam usados na maioria dos casos. O formato de um endereço é o seguinte: $BBHHDD.

* BB é o "byte do banco";
* HH é o "High byte" ou Byte mais significativo;
* DD é o "Low byte" ou Byte menos significativo.

Os endereços podem ser escritos de 3 maneiras: $BBHHDD, $HHDD e $DD, como $7E0003, $0003 e $03.

* $DD são os “endereços de página direta”
* $HHDD são os “endereços absolutos”
* $BBHHDD são os “endereços longos”

Conforme estabelecido anteriormente, um endereço pode conter apenas um *byte*. Se você acessar um determinado endereço no modo 16 *bits*, significa que você realmente está acessando o "endereço" e "endereço + 1", porque um número de 16 *bits* consiste em dois *bytes*.

A figura a seguir temos uma visão geral da memória básica do SNES \(também conhecido como mapa de memória\):

![O Mapa de Memória LoROM](/home/sandro/Documentos/snes-assembly-book/make-pdf/.gitbook/assets/memory.png)

Este mapa de memória está no formato "LoROM". Se você é um hacker de SMW, não precisa se preocupar com o que isso significa; apenas considere este mapa de memória por garantia.

## ROM

*ROM* significa "Read-Only Memory" e é exatamente isso: uma memória que só pode ser lida. Isso significa que você não pode alterar a *ROM* armazenando valores nela com o *ASM*. Você pode dizer que é o próprio jogo ou programa, que contém todo o código *ASM* e tabelas de dados, bem como recursos como gráficos, música e assim por diante. Alternativamente: é o arquivo .smc/.sfc/.fig/etc. que você carrega em emuladores.

## RAM

*RAM* significa "Random-Access Memory". Esta é a memória dinâmica que permite que qualquer coisa seja escrita nela a qualquer momento. Você poderia dizer que este é o lugar onde você tem as variáveis que são importantes e têm significado. A *RAM* pode ser gravada para obter um certo efeito. Por exemplo, se você escrever $04 nas vidas extras do jogador, e ele terá exatamente 4 vidas extras.

A *RAM* do SNES tem o tamanho 128kB e está localizado nos endereços de $7E0000 a $7FFFFF. A *RAM* é totalmente genérica. Não existe uma regra como “o endereço $7E0120 é usado para as vidas do jogador em todos os jogos SNES.” Você mesmo define a finalidade da *RAM*, escrevendo seu próprio código *ASM*.

O mapa de memória mostra que os bancos $00-3F contêm um "espelho" da *RAM*. Os endereços de *RAM* espelhados são endereços que contêm o mesmo valor em todos os bancos. Isso significa que o endereço de *RAM* $001234 contém exatamente o mesmo valor de $0F1234 em todos os momentos. Ter a *RAM* espelhada significa que o código em execução na ROM nesses bancos pode acessar a *RAM* de $7E0000 a $7E1FFF com mais "facilidade". Por outro lado, o código executado nos bancos $40-6F tem mais problemas para acessar a *RAM* porque a *RAM* não é espelhada nesse local.

Por razões de simplicidade, você **sempre** pode assumir que o banco $00 é igual ao banco $7E.

## SRAM

*SRAM* significa "Static Random-Access Memory". Também tem o tamanho de 128kB e está localizado em blocos de 32kB entre $700000 a $707FFF, $710000 a $717FFF, $720000 a $727FFF e $730000 a $737FFF, embora o tamanho final da *SRAM* dependa das próprias especificações de *ROM*, graças a algo chamado "cabeçalho interno da ROM". a *SRAM* não é espelhada em outros bancos.

*SRAM* se comporta exatamente como *RAM*; você pode armazenar e carregar qualquer coisa nela, mas os valores não são apagados quando o SNES é reiniciado. A memória *SRAM* é mantida viva por uma bateria que está presente em um cartucho de SNES. Quando a bateria se esgota ou é removida, a *SRAM* não funcionará corretamente e possivelmente perderá os dados após cada reinicialização. Nos emuladores, a *SRAM* é armazenada nos arquivos ".srm".

*SRAM* é geralmente usado para salvar arquivos, embora também possa ser usada como uma memória *RAM* extra.

# Os Registradores do SNES

O SNES possui vários “registradores” que são usados para diferentes finalidades. Não podemos nos esquecer deles; são uma das razões pelas quais possibilitam o SNES funcionar corretamente.. Basicamente, registradores são “variáveis globais” que podem ser usados para armazenar valores, ou podem ser usados em operações matemáticas, lógica e todas aquelas coisas sofisticadas! Esses registradores podem ser acessados a qualquer momento.

## Acumulador (A)

O acumulador, também conhecido como **A**, é usado para operações matemáticas em geral, deslocamento de bits, operações bit a bit e carregamento de valores indiretos. A também pode conter variáveis ​​de uso geral para armazenar valores na memória e em outros registradores. Este registrador pode conter um valor de 8 ou 16-bit

O acumulador às vezes é referido como `B` ou `C` em alguns opcodes. B significa o high byte do acumulador, enquanto C significa o acumulador completo de 16-bit.

{% hint style = "aviso"%}
Na verdade, esse registrador pode ser sempre considerado como de 16-bit. Quando A está no modo de 8-bit, você acessa o low byte desse registrador. Quando A está no modo de 16-bit, você acessa o ambos high e low byte desse registrador ao mesmo tempo. O high byte não é apagado quando A entra no modo de 8-bit, mesmo quando novos valores são gravados em A, razão pela qual o high byte pode ser considerado "oculto". Além disso, certas instruções usam high e low bytes do registrador A, independentemente de A estar no modo de 8 ou 16-bit.
{% endhint%}

## Indexadores (X,Y)

Os indexadores são dois registradores, conhecidos como **X** e **Y**. Embora sejam registradores separados, eles têm exatamente as mesmas finalidades e se comportam exatamente da mesma forma. Esses registradores são feitos para indexação, explicada posteriormente neste tutorial. Esses registradores também podem ser de 8 ou 16-bit. X e Y também podem conter variáveis ​​de uso geral para armazenar valores na memória e em outros registradores.

X e Y são “interligados” - e só podem estar no modo de 8 ou 16-bit ao mesmo tempo. Um deles não pode ser de 8-bit e o outro de 16-bit.

{% hint style = "aviso"%}
Quando X e Y saem do modo de 16-bit, seus high bytes são zerados para o valor $00, ao contrário do registrador A, onde o high byte permanece intacto.
{% endhint%}

## Direct page (D)

O registrador de Direct page é um registrador de 16-bit, usado no  modo de endereçamento de Direct page (explicado posteriormente neste tutorial). Quando você acessa um endereço da memória pela notação de Direct Page, o valor da Direct Page atual é adicionado nesse endereço. Geralmente, você pode ignorar esse registrador se estiver apenas iniciando em assembly.

## Stack Pointer (SP)

O stack pointer é um registrador de 16-bit que mantém o ponteiro do stack na RAM (explicado mais tarde neste tutorial), relativo ao endereço de memória $000000. O registrador muda dinamicamente, conforme você adiciona e requisita valores na stack (explicado posteriormente no tutorial).

## Processor Status (P)

O registrador de status do processador contém os sinalizadores do processador atual no formato de 8-bit. Existem 8 sinalizadores de processador e todos ocupam um bit. Alterar esse registrador alteraria muito o comportamento do SNES. Os sinalizadores do processador são explicados posteriormente neste tutorial.

## Data bank (DB)

O registrador de data bank  contém um único byte do endereço do data bank atual. Quando você acessa um endereço usando a notação de "endereço absoluto", o SNES usará esse registrador para determinar o banco do endereço.

## Program bank (PB)

Este registrador contém o primeiro byte do endereço instrução do bank atual que será executada no momento. Assim, se houver um código executado no endereço $018009, este registrador terá valor $01.

## Program counter (PC)

Este registrador contém os high e low bytes do endereço da instrução que será executada no momento. Portanto, se houver uma instrução executada em $018009, este registrador terá o valor $8009.