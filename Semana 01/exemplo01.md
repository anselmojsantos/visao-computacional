# exemplo01.ipynb — Fundamentos de imagem digital

Este cadastro explica o notebook `exemplo01.ipynb` da pasta `Semana 01`. Ele é uma aula prática sobre **como a imagem digital é formada, percebida e representada**: luz, cor, brilho, contraste, formação da imagem, amostragem, quantização, resolução e custo de memória.

Todos os conceitos são demonstrados com código executável: cada célula gera uma figura ou uma tabela que ilustra o ponto teórico correspondente.

## Como executar

- O notebook usa `numpy`, `opencv-python` (`cv2`), `matplotlib`, `scikit-image` (`skimage`) e `Pillow`.
- A célula 1 carrega a imagem com o caminho relativo `..\img\img1.JPEG`. Por isso, execute o notebook **de dentro da pasta `Semana 01`**; a imagem está em `img/img1.jpeg`, na raiz do projeto.
- A célula 17 usa o caminho absoluto `C:\TI\Python\Visão Computacional\img\img1.JPEG` apenas para diagnosticar problemas de carregamento. Ajuste esse caminho se o projeto estiver em outro local.
- A partir da célula 8, o notebook troca a foto por `skimage.data.camera()`, uma imagem de teste embutida na biblioteca, para que os experimentos seguintes não dependam de arquivos externos.

## Estrutura do notebook

### 1. Preparação e leitura da imagem (células 0–2)

A célula 0 importa as bibliotecas e ativa `%matplotlib inline`, para que as figuras apareçam dentro do notebook.

A célula 1 lê `img1.JPEG` com OpenCV e converte de BGR para RGB:

```python
img_rgb = cv2.cvtColor(cv2.imread(adress), cv2.COLOR_BGR2RGB)
```

Essa conversão é necessária porque o OpenCV lê canais na ordem azul-verde-vermelho, enquanto o `matplotlib` espera vermelho-verde-azul. A saída impressa confirma as propriedades da imagem:

- formato `(1600, 1200, 3)` → 1600 linhas, 1200 colunas, 3 canais de cor;
- `dtype uint8` → cada intensidade vai de 0 a 255;
- `min/max = 0 / 255` → a faixa completa de intensidade está ocupada.

A célula 2 apenas exibe a imagem de entrada.

### 2. Canais de cor e luminância (célula 3)

A imagem é separada em `red`, `green` e `blue`, cada um exibido com seu mapa de cores. Em seguida, `color.rgb2gray(img_rgb)` calcula o cinza como uma média ponderada dos canais, com pesos próximos a `0,21 R + 0,72 G + 0,07 B`. O verde domina porque o olho humano é mais sensível a ele.

O resultado vem em ponto flutuante no intervalo `[0, 1]`; a linha `(img_gray * 255).astype(np.uint8)` o converte para `uint8` para exibição e para o cálculo do histograma.

> Detalhe: são criados 6 painéis com `plt.subplots(1, 6, ...)`, mas apenas os 5 primeiros são preenchidos. O último aparece vazio.

### 3. Contraste simultâneo (célula 4)

Um mesmo quadrado cinza (valor 128) é colocado sobre um fundo escuro (20) e sobre um fundo claro (220). Ele parece **mais claro no fundo escuro e mais escuro no fundo claro**, embora os valores numéricos sejam idênticos.

É a demonstração de que o brilho percebido depende do contexto, não apenas do valor do pixel. O título do gráfico cita "I=30" para o fundo escuro, mas o código usa 20.

### 4. Bordas como transição de intensidade (célula 5)

Cria-se um degrau em `x > 0.5` e ele é suavizado por uma média móvel de 9 amostras (`np.convolve` com `np.ones(9)/9`). O gráfico mostra o perfil de intensidade e a imagem sintética resultante.

Uma borda, do ponto de vista do processamento, é uma mudança de intensidade ao longo do espaço; suavizar o perfil alarga essa transição.

### 5. Histograma e equalização (célula 6)

A célula mostra três coisas lado a lado: a imagem em cinza, seu histograma (contagem de pixels por intensidade) e o resultado de `exposure.equalize_hist`.

A equalização redistribui os valores usando a função de distribuição acumulada do histograma, espalhando intensidades concentradas e aumentando o contraste global. A função retorna floats entre 0 e 1.

### 6. Energia dos fótons e espectro visível (célula 7)

A célula aplica a relação de Planck:

$$E = \frac{h\,c}{\lambda}$$

com `h = 6,626×10⁻³⁴ J·s`, `c = 3,0×10⁸ m/s` e comprimentos de onda de 380 a 750 nm. O gráfico sobrepõe faixas coloridas do visível.

Como `E` é inversamente proporcional a `λ`, a luz violeta (380 nm, ≈ 5,23×10⁻¹⁹ J) carrega mais energia por fóton que a vermelha (750 nm, ≈ 2,65×10⁻¹⁹ J).

### 7. Modelo de formação da imagem (células 8–9)

Aqui a imagem de trabalho passa a ser `data.camera()` (512×512), convertida para float em `[0, 1]`.

A célula 9 implementa o modelo clássico:

$$f(x,y) = i(x,y) \cdot r(x,y)$$

- `r` é a **reflectância** da cena: uma faixa escura (0,05), uma região clara (0,80) e outra mais clara ainda (0,93);
- `i` é a **iluminação**: 90.000 no sol e 10.000 em dia nublado;
- `f` é a imagem captada.

A função `norm255` faz normalização min-max antes de exibir. Como `f_sol` e `f_nublado` são a mesma `r` multiplicada por constantes diferentes, a normalização remove exatamente a diferença de iluminação e as duas imagens aparecem iguais. A lição implícita é importante: **normalizar apaga informação de brilho absoluto**, preservando apenas o contraste relativo.

> Atenção: no `zip` que desenha os dois painéis, só foi fornecido um título. O `zip` para na primeira combinação e o segundo painel fica em branco.

### 8. Amostragem (célula 10)

`transform.downscale_local_mean(img_f, (p, p))` reduz a grade de pixels com passos `p = 1, 4, 16, 64`. Os títulos mostram as dimensões resultantes. Quanto maior o passo, menor a imagem e mais visível a **pixelização**: a amostragem define a resolução espacial.

### 9. Quantização (célula 11)

```python
L = 2**k
q = np.floor(img_f * (L - 1) + 0.5) / (L - 1)
```

Cada pixel é arredondado para um de `L = 2^k` níveis. A célula compara `k = 8, 6, 4, 3, 2, 1`, de 256 até apenas 2 níveis. Com poucos níveis surgem **falsos contornos** (*banding*): áreas que deveriam ser contínuas viram faixas uniformes.

### 10. Digitalização completa (célula 12)

`digitalize` combina as duas etapas: primeiro amostra, depois quantiza. Quatro combinações de `(passo, k)` mostram o compromisso entre detalhe espacial e riqueza de tons: `(1, 8)`, `(4, 4)`, `(16, 2)` e `(8, 3)`.

### 11. DPI e tamanho físico (célula 13)

`simula_dpi` mantém a mesma área física (2,95 polegadas) e muda apenas a resolução: 1250, 300, 150 e 75 dpi. O número de pixels é `dpi × polegadas`. Menos dpi significa menos pixels para a mesma cena, com perda de detalhe.

### 12. Curvas de isopreferência (célula 14)

A célula plota `k = 2·log₂(C/N)` para três níveis de complexidade de cena (`C = 2×10⁴`, `5×10⁴` e `1,2×10⁵`), com `N` de 64 a 1024.

As curvas mostram a troca entre resolução espacial (`N`) e profundidade de bits (`k`): ao reduzir o número de pixels, é possível aumentar o número de níveis de cinza mantendo uma qualidade percebida semelhante — e vice-versa.

### 13. Custo de memória (célula 15)

A tabela calcula `B = N² · K` bits para `N` de 32 a 8192 e `K` de 1 a 8. Exemplos:

| Imagem | Bits | Memória |
|---|---:|---:|
| 512×512, 1 bit/pixel | 262.144 | 0,031 MiB |
| 512×512, 8 bits/pixel | 2.097.152 | 0,25 MiB |
| 8192×8192, 8 bits/pixel | 536.870.912 | 64 MiB |

O cabeçalho impresso diz `end=` onde deveria dizer `N=`; é apenas um erro de digitação na legenda.

### 14. Geometria do sensor (célula 16)

Duas funções simulam formas diferentes de captura com `passo = 16`:

- `varredura_linha` preserva uma fileira completa a cada 16 linhas — sensor de linha;
- `varredura_ponto` preserva um ponto a cada 16 posições em x e em y — sensor pontual.

O que o sensor consegue captar por vez limita a amostragem. O sensor de linha mantém mais informação horizontal; o pontual deixa uma malha esparsa.

### 15. Diagnóstico de caminho (célula 17)

Verifica se a pasta e o arquivo `img1.JPEG` existem, lista os arquivos da pasta e confirma se `cv2.imread` carregou a imagem. É uma célula de depuração para o erro mais comum em notebooks: caminho relativo errado, especialmente com acentos (`Visão Computacional`).

A célula 18 está vazia.

## Resumo dos conceitos

| Células | Tema | Ideia central |
|---|---|---|
| 0–2 | Leitura | OpenCV usa BGR; imagem `uint8`, 0–255 |
| 3 | Cor | Cinza é média ponderada dos canais |
| 4 | Percepção | Contraste simultâneo altera o brilho percebido |
| 5 | Bordas | Borda é transição de intensidade |
| 6 | Histograma | Equalização melhora o contraste global |
| 7 | Luz | `E = hc/λ`; quanto menor `λ`, maior `E` |
| 8–9 | Formação | `f = i · r`; iluminação vs. reflectância |
| 10 | Amostragem | Menos pixels → pixelização |
| 11 | Quantização | Menos níveis → falsos contornos |
| 12–14 | Compromisso | DPI e curvas de isopreferência |
| 15–16 | Limites práticos | Memória e geometria do sensor |
| 17 | Depuração | Conferir caminhos antes de processar |

## Observações sobre o código

Alguns detalhes do notebook são erros de digitação ou pequenos descuidos que não atrapalham a compreensão, mas vale registrá-los:

- `print("Shap", ...)` em vez de `Shape` (célula 1);
- `Canal Rede` em vez de `Red` e `Greens`/`Blues` no plural (célula 3);
- painel extra vazio na grade de canais (célula 3);
- fundo escuro descrito como `I=30`, mas criado com valor 20 (célula 4);
- `degrau ideial` e `pexels` (células 5 e 6);
- `Após equlização` (célula 6);
- segundo painel em branco por falta de título no `zip` (célula 9);
- `end=` no cabeçalho da tabela de memória (célula 15);
- variável `img_rgb` usada para uma imagem em cinza a partir da célula 8.
