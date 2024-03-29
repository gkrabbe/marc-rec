## handRecognition
reconhecimento de mão

## master recognition
reconhecimento de pessoas e outros

## opencv_createsamples 

Cria amostras positivas de uma única imagem de objeto ou coleção de imagens positivas. O esquema de criação de amostras de teste é semelhante ao treinamento de criação de amostras, pois cada amostra de teste é uma imagem de fundo na qual uma distorção aleatória e escala aleatória. A instância da imagem do objeto é colada em uma posição aleatória.

## opencv_annotation 

A ferramenta opencv_annotations é muito útil para ajudá-lo a capturar todas as coordenadas retangulares de todas as amostras positivas que você gostaria de usar em seu treinamento em cascata. Coordenada do retângulo significa (x, y, w, h). O programa irá gerar um arquivo de texto que contém o caminho do arquivo para a imagem positiva, o número de imagens (a contagem é geralmente 1) e as coordenadas do retângulo significam (x, y, w, h).

## opencv_traincascade

Serve para treinar classificadores no modelo cascata.

# TUTORIAL DE COMO TREINAR UMA REDE COM CV2

O tutorial atual percorrerá todas as diferentes etapas: coleta de dados de treinamento, preparação dos dados de treinamento e execução do treinamento real do modelo.

Para dar suporte a este tutorial, vários aplicativos oficiais do OpenCV serão usados: opencv_createsamples , opencv_annotation , opencv_traincascade. 

### Preparação dos dados de treinamento

Para treinar uma cascata aprimorada de classificadores fracos, precisamos de um conjunto de amostras positivas (contendo objetos reais que você deseja detectar) e um conjunto de imagens negativas (contendo tudo que você não deseja detectar). O conjunto de amostras negativas deve ser preparado manualmente, enquanto o conjunto de amostras positivas é criado usando o aplicativo opencv_createsamples.

## Amostras negativas

Amostras negativas são obtidas de imagens arbitrárias, não contendo objetos que você deseja detectar. Essas imagens negativas, das quais as amostras são geradas, devem ser listadas em um arquivo de imagem negativa especial contendo um caminho de imagem por linha.

Cada imagem deve ser igual ou maior que o tamanho da janela de treinamento desejado (que corresponde às dimensões do modelo, na maioria das vezes sendo o tamanho médio do seu objeto), porque essas imagens são usadas para subamostrar uma determinada imagem negativa em várias imagens.

Um exemplo de um arquivo de descrição negativo:

### Estrutura de diretório:

/ img

img1.jpg

img2.jpg

bg.txt

Arquivo bg.txt:

img / img1.jpg

img / img2.jpg

Seu conjunto de amostras de janelas negativas será usado para informar a etapa de aprendizado de máquina, aumentando nesse caso o que não procurar ao tentar encontrar seus objetos de interesse.

## mostras positivas

Amostras positivas são criadas pelo aplicativo opencv_createsamples. Eles são usados pelo processo de otimização para definir o que o modelo deve realmente procurar ao tentar encontrar seus objetos de interesse. O aplicativo suporta duas maneiras de gerar um conjunto de dados de amostra positivo.

1.	Você pode gerar vários pontos positivos a partir de uma única imagem de objeto positivo.
2.	Você mesmo pode fornecer todos os aspectos positivos e usar a ferramenta apenas para cortá-los, redimensioná-los e colocá-los no formato binário necessário.

A primeira abordagem pega uma imagem de objeto único com, por exemplo, um logotipo da empresa e cria um grande conjunto de amostras positivas a partir da imagem de objeto determinada, girando aleatoriamente o objeto, alterando a intensidade da imagem e colocando a imagem em fundos arbitrários. A quantidade e o intervalo de aleatoriedade podem ser controlados pelos argumentos da linha de comandos do aplicativo opencv_createsamples.

Argumentos da linha de comando:

- -vec <vec_file_name> : Nome do arquivo de saída que contém as amostras positivas para treinamento.
- -img <image_file_name> : Imagem do objeto de origem (por exemplo, um logotipo da empresa).
- -bg <background_file_name>: Arquivo de descrição de plano de fundo; contém uma lista de imagens usadas como plano de fundo para versões distorcidas aleatoriamente do objeto.
- -num <number_of_samples> : Número de amostras positivas a serem geradas.
- -bgcolor <background_color>: Cor do plano de fundo (atualmente são assumidas imagens em escala de cinza); a cor de fundo denota a cor transparente. Como pode haver artefatos de compactação, a quantidade de tolerância de cores pode ser especificada por -bgthresh. Todos os pixels dentro do intervalo bgcolor-bgthresh e bgcolor + bgthresh são interpretados como transparentes.
- -bgthresh <background_color_threshold>
- -inv : Se especificado, as cores serão invertidas.
- -randinv : Se especificado, as cores serão invertidas aleatoriamente.
- -maxidev <max_intensity_deviation> : Desvio de intensidade máxima de pixels em amostras em primeiro plano.
- -maxxangle <max_x_rotation_angle> : O ângulo máximo de rotação em direção ao eixo x deve ser dado em radianos.
- -maxyangle <max_y_rotation_angle> : O ângulo máximo de rotação em direção ao eixo y deve ser dado em radianos.
- -maxzangle <max_z_rotation_angle> : O ângulo de rotação máximo em direção ao eixo z deve ser dado em radianos.
- -show: Opção de depuração útil. Se especificado, cada amostra será mostrada. Pressionar Esc continuará o processo de criação de amostras sem mostrar cada amostra.

- -w <sample_width> : Largura (em pixels) das amostras de saída.
- -h <sample_height> : Altura (em pixels) das amostras de saída.

Ao executar opencv_createsamples dessa maneira, o procedimento a seguir é usado para criar uma instância de objeto de amostra. A imagem de origem fornecida é girada aleatoriamente em torno dos três eixos. O ângulo escolhido é limitado por -maxxangle, -maxyanglee -maxzangle. Em seguida, pixels com a intensidade do [bg_color-bg_color_threshold; bg_color + bg_color_threshold] são interpretados como transparentes. O ruído branco é adicionado às intensidades do primeiro plano. Se a -invchave for especificada, as intensidades de pixel em primeiro plano serão invertidas. Se a -randinvchave for especificada, o algoritmo seleciona aleatoriamente se a inversão deve ser aplicada a esta amostra. Por fim, a imagem obtida é colocada em um plano de fundo arbitrário a partir do arquivo de descrição de plano de fundo, redimensionada para o tamanho desejado especificado por -we-he armazenado no arquivo vec, especificado pela -vec opção da linha de comandos.

Amostras positivas também podem ser obtidas de uma coleção de imagens previamente marcadas, que é a maneira desejada ao criar modelos de objetos robustos. Essa coleção é descrita por um arquivo de texto semelhante ao arquivo de descrição de plano de fundo. Cada linha deste arquivo corresponde a uma imagem. O primeiro elemento da linha é o nome do arquivo, seguido pelo número de anotações de objetos, seguido pelos números que descrevem as coordenadas dos objetos que delimitam os retângulos (x, y, largura, altura).

### Um exemplo de arquivo de descrição:

Estrutura de diretório:
/ img
img1.jpg
img2.jpg
info.dat
Arquivo info.dat:
img / img1.jpg 1 140 100 45 45
img / img2.jpg 2 100 200 50 50 50 30 25 25
A imagem img1.jpg contém uma instância de objeto único com as seguintes coordenadas do retângulo delimitador: (140, 100, 45, 45). A imagem img2.jpg contém duas instâncias do objeto.

Para criar amostras positivas dessa coleção, o –info argumento deve ser especificado em vez de -img:

- -info <collection_file_name> : Arquivo de descrição da coleção de imagens marcadas.
Observe que, neste caso, parâmetros como -bg, -bgcolor, -bgthreshold, -inv, -randinv, -maxxangle, -maxyangle, -maxzanglesão simplesmente ignorados e não são mais usados. O esquema de criação de amostras neste caso é o seguinte. As instâncias do objeto são obtidas das imagens fornecidas, cortando as caixas delimitadoras fornecidas das imagens originais. Em seguida, eles são redimensionados para atingir o tamanho das amostras (definidas por -we -h) e armazenados no arquivo vec de saída, definido pelo -vecparâmetro Nenhuma distorção é aplicada, de modo que os argumentos afetando somente são -w, -h, -showe -num.

O processo manual de criação do –info arquivo também pode ser realizado usando a ferramenta opencv_annotation. Esta é uma ferramenta de código aberto para selecionar visualmente as regiões de interesse de suas instâncias de objeto em qualquer imagem.

### Observações extras

- O utilitário opencv_createsamples pode ser usado para examinar amostras armazenadas em qualquer arquivo de amostras positivas. Para fazer isso apenas -vec, -we os -hparâmetros devem ser especificados.

Usando a ferramenta de anotação integrada do OpenCV
Desde o OpenCV 3.x, a comunidade fornece e mantém uma ferramenta de anotação de código aberto, usada para gerar o -infoarquivo. A ferramenta pode ser acessada pelo comando opencv_annotation se os aplicativos OpenCV forem construídos.

O uso da ferramenta é bastante direto. A ferramenta aceita vários parâmetros obrigatórios e alguns opcionais:

- --annotations (obrigatório): caminho para o arquivo txt de anotações, no qual você deseja armazenar suas anotações, que são passadas para o -infoparâmetro [exemplo - /data/annotations.txt]
- --images (obrigatório): caminho para a pasta que contém as imagens com seus objetos [exemplo - / data / testimages /]
- --maxWindowHeight (opcional): se a imagem de entrada for maior em altura, em seguida, a resolução fornecida aqui, redimensione a imagem para facilitar a anotação usando --resizeFactor.
- --resizeFactor (opcional): fator usado para redimensionar a imagem de entrada ao usar o --maxWindowHeightparâmetro.

Observe que os parâmetros opcionais podem ser usados apenas juntos. Um exemplo de comando que pode ser usado pode ser visto abaixo
opencv_annotation --annotations = / caminho / para / anotações / arquivo.txt --images = / caminho / para / imagem / pasta /
Este comando abrirá uma janela contendo a primeira imagem e o cursor do mouse, que serão usados para anotação. Basicamente, existem várias teclas que acionam uma ação. O botão esquerdo do mouse é usado para selecionar o primeiro canto do seu objeto, depois continua desenhando até você ficar bem e para quando um segundo clique no botão esquerdo do mouse é registrado. Após cada seleção, você tem as seguintes opções:

- Pressionando c: confirme a anotação, deixando a anotação verde e confirmando que está armazenada
- Pressionando d: exclua a última anotação da lista de anotações (fácil para remover anotações erradas)
- Pressionando n: continue para a próxima imagem
- Pressionando ESC: isso sairá do software de anotação

Finalmente, você terminará com um arquivo de anotação utilizável que pode ser passado ao –info argumento do opencv_createsamples.

## Treinamento em cascata

O próximo passo é o treinamento real da cascata, com base no conjunto de dados positivo e negativo que foi preparado anteriormente.

Argumentos de linha de comando do aplicativo opencv_traincascade agrupados por finalidades:

Argumentos comuns:

- -data <cascade_dir_name>: Onde o classificador treinado deve ser armazenado. Esta pasta deve ser criada manualmente com antecedência.
- -vec <vec_file_name> : arquivo vec com amostras positivas (criadas pelo utilitário opencv_createsamples).
- -bg <background_file_name>: Arquivo de descrição de plano de fundo. Este é o arquivo que contém as imagens de amostra negativas.
- -numPos <number_of_positive_samples> : Número de amostras positivas usadas no treinamento para cada estágio do classificador.
- -numNeg <number_of_negative_samples> : Número de amostras negativas usadas no treinamento para cada estágio do classificador.
- -numStages <number_of_stages> : Número de estágios em cascata a serem treinados.

- -precalcValBufSize <precalculated_vals_buffer_size_in_Mb>: Tamanho do buffer para valores de recursos pré-calculados (em Mb). Quanto mais memória você atribuir mais rápido o processo de formação, no entanto ter em mente que -precalcValBufSizee -precalcIdxBufSizecombinado não deve exceder você memória disponível no sistema.
- -precalcIdxBufSize <precalculated_idxs_buffer_size_in_Mb>: Tamanho do buffer para índices de recursos pré-calculados (em Mb). Quanto mais memória você atribuir mais rápido o processo de formação, no entanto ter em mente que -precalcValBufSizee -precalcIdxBufSizecombinado não deve exceder você memória disponível no sistema.
- -baseFormatSave: Esse argumento é real no caso de recursos do tipo Haar. Se for especificado, a cascata será salva no formato antigo. Isso está disponível apenas por razões de compatibilidade com versões anteriores e para permitir que os usuários presos à interface obsoleta antiga treinem pelo menos modelos usando a interface mais recente.
- -numThreads <max_number_of_threads>: Número máximo de threads a serem usados durante o treinamento. Observe que o número real de threads usados pode ser menor, dependendo da sua máquina e das opções de compilação. Por padrão, o máximo de threads disponíveis é selecionado se você criou o OpenCV com suporte a TBB, o que é necessário para essa otimização.
- -acceptanceRatioBreakValue <break_value>: Esse argumento é usado para determinar quão preciso seu modelo deve continuar aprendendo e quando parar. Uma boa orientação é treinar não mais do que 10e-5, para garantir que o modelo não exagere nos dados de treinamento. Por padrão, esse valor é definido como -1 para desativar esse recurso.
- Parâmetros em cascata:
- -stageType <BOOST(default)>: Tipo de etapas. No momento, apenas classificadores aprimorados são suportados como um tipo de estágio.
- -featureType<{HAAR(default), LBP}> : Tipo de recursos: HAAR - recursos semelhantes ao Haar, LBP - padrões binários locais.
- -w <sampleWidth>: Largura das amostras de treinamento (em pixels). Deve ter exatamente o mesmo valor usado durante a criação de amostras de treinamento (utilitário opencv_createsamples).
- -h <sampleHeight>: Altura das amostras de treinamento (em pixels). Deve ter exatamente o mesmo valor usado durante a criação de amostras de treinamento (utilitário opencv_createsamples).
- Parâmetros de classificador aprimorados:
- -bt <{DAB, RAB, LB, GAB(default)}> : Tipo de classificadores impulsionados: DAB - AdaBoost discreto, RAB - AdaBoost real, LB - LogitBoost, GAB - AdaBoost suave.
- -minHitRate <min_hit_rate>: Taxa de acerto mínima desejada para cada estágio do classificador. A taxa geral de acertos pode ser estimada em (min_hit_rate ^ number_of_stages), [213] §4.1.
- -maxFalseAlarmRate <max_false_alarm_rate>: Taxa máxima de alarmes falsos desejada para cada estágio do classificador. A taxa geral de alarmes falsos pode ser estimada em (max_false_alarm_rate ^ number_of_stages), [213] §4.1.
- -weightTrimRate <weight_trim_rate>: Especifica se o corte deve ser usado e seu peso. Uma escolha decente é 0,95.
- -maxDepth <max_depth_of_weak_tree>: Profundidade máxima de uma árvore fraca. Uma escolha decente é 1, que é o caso de tocos.
- -maxWeakCount <max_weak_tree_count>: Contagem máxima de árvores fracas para cada estágio em cascata. O classificador impulsionado (estágio) terá tantas árvores fracas (<= maxWeakCount), conforme necessário para alcançar o dado -maxFalseAlarmRate.

Depois que o aplicativo opencv_traincascade terminar seu trabalho, a cascata treinada será salva em um cascade.xmlarquivo na –datapasta. Outros arquivos nesta pasta são criados para o caso de treinamento interrompido, portanto, você pode excluí-los após a conclusão do treinamento.

O treinamento está concluído e você pode testar seu classificador em cascata!

