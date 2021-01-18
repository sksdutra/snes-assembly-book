# O Livro de Assembly para SNES

# Sobre a Tradução

A tradução e revisão deste documento é um esforço coletivo de membros da cena de ROM hacking brasileira, são eles: Gambas, Israel, Joapeer e Ondinha, com o único intuito de prover documentação de qualidade e em português a todos que tenham interesse no SNES/65c816 no Brasil. Esperamos que seja útil em seu aprendizado!

# Sumário

* [Introdução](#Introdução)
* [Iniciando](#Iniciando)
* [Contribuindo](#Contribuindo)

## O Básico

* [Hexadecimal](#Hexadecimal)
* [Binário](the-basics/binary.md)
* [Memória](the-basics/memory.md)
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