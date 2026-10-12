# [**Cidade** (link para o jogo)](https://cidade.do.not.br) — simulador de cidade e transporte em Rust

Simulador urbano 3D inspirado em Cities: Skylines. A simulação, a câmera e toda
a geometria 3D são feitas em Rust e rodam no navegador via WebAssembly; a página
usa WebGL2 só para enviar os buffers prontos à placa de vídeo.

## Como publicar (sem compilar nada)

A pasta `dist/` já vem pronta. O jogo é uma página única (`index.html`), com o
WebAssembly embutido, e funciona em qualquer hospedagem de site estático.

- **Testar no computador:** dê dois cliques em `dist/index.html`, ou rode
  `python -m http.server 8000` dentro de `dist/` e abra http://localhost:8000
- **Netlify:** acesse https://app.netlify.com/drop e arraste a pasta `dist/`.
- **GitHub Pages:** envie o projeto para um repositório, coloque `index.html` na
  raiz ou em `/docs` e ative em *Settings → Pages*.
- **Hospedagem própria:** envie `dist/index.html` para a pasta pública (`public_html`, `www`).

Precisa de um navegador com WebGL2 (Chrome, Edge, Firefox e Safari atuais).

## Como compilar

```bash
rustup target add wasm32-unknown-unknown   # uma vez só
./build.sh                                  # gera dist/index.html
cargo test --release -- --nocapture         # simula 1 ano no mapa de 2000×2000
```

O projeto não usa nenhuma crate externa.

## Controles

| Ação | Mouse / teclado | Toque |
|---|---|---|
| Construir | arrastar com o botão esquerdo | arrastar com um dedo |
| Via em curva | **Curva** no painel de vias (ou tecla C): clique no começo, no ponto que puxa a curva e no fim | três toques |
| Via livre (spline) | **Livre** no painel de vias (tecla C alterna): clique os pontos de controle; Enter ou duplo clique termina, Backspace desfaz o último | toques e toque duplo |
| Altura da via (viaduto, rampa, túnel) | Page Up / Page Down, ou − / + no painel | botões − / + |
| Largura, faixas e calçadas | **Perfil da via** no painel de vias | — |
| Editar vias (mover nó/curva, inserir, unir, paralela, rotatória, nível) | **Editar** no painel de vias, ou tecla V | arrastar / tocar |
| Trocar o tipo de uma via | com a ferramenta da via nova, clique na via existente | tocar na via |
| Cancelar a via | Esc | — |
| Mover o mapa | botão direito, ou ferramenta **Mover** (0), ou WASD/setas | ferramenta Mover |
| Zoom | roda do mouse (aproxima no ponto do cursor), + e − | pinça com dois dedos |
| Girar | Q / E, ou botões ⟲ ⟳ | botões ⟲ ⟳ |
| Ir para outro lugar | clicar no minimapa | tocar no minimapa |
| Pausar | espaço | botão Pausa |

## Estrutura

| Arquivo | O que faz |
|---|---|
| `src/sim.rs` | Mapa 2000×2000, construção, zonas, demanda, valor do terreno, energia e água, orçamento, veículos |
| `src/net.rs` | **Rede de vias livre**: os 12 tipos de via e trilho, nós, trechos em curva (Bézier), faixas, calçadas e rotas A* pela rede |
| `src/roads.rs` | Construir, cortar, emendar, trocar o tipo e demolir vias; ferramentas reta e curva; conversão de cidades antigas |
| `src/traffic.rs` | **Trânsito por faixa**: filas por faixa e calçada, ninguém ocupa o mesmo espaço, cruzamentos com semáforo ou vez, trilho simples |
| `src/roadgfx.rs` | Malha 3D das vias: pistas, calçadas, meio-fio, canteiros, pontes, trilhos, cruzamentos (cantos arredondados, transição de largura), bocas de túnel, postes e árvores; tesselação adaptativa |
| `src/roadedit.rs` | **Sistema viário**: traçado livre por pontos de controle (spline com tangentes automáticas), bifurcação em Y, mover nós e curvas, inserir ponto, unir, juntar nós, nível do nó, via paralela, rotatória e validação. Veja `docs/SISTEMA_VIARIO.md` |
| `src/path.rs` | Rotas A* por quadra (metrô e serviços que não usam a rede) |
| `src/citizens.rs` | **Cidadãos**: cada pessoa com casa, trabalho, dinheiro, necessidades, rotina, doença, crime, idade e escolha de transporte |
| `src/megacity.rs` | Metrópole de teste com 1 milhão de moradores |
| `src/services.rs` | Polícia, bombeiros e saúde: cobertura, crime, incêndios, viaturas e ambulâncias |
| `src/transit.rs` | Linhas de ônibus, metrô e trem, veículos e escolha entre carro e transporte público |
| `src/disasters.rs` | Monstro, invasão, terremoto, tornado, ruínas e defesa (base militar, tanques, helicópteros) |
| `src/save.rs` | Salvar e carregar: semente do terreno + só as quadras que mudaram |
| `src/stats.rs` | Histórico diário de indicadores (últimos 2 anos) para os gráficos |
| `src/gfx.rs` | Blocos de 32×32 com listas por nível de detalhe, área visível, veículos e alertas por quadro, minimapa |
| `src/models.rs` | **Todos os modelos estáticos** (prédios de zona seguem o nível do lote; pegadas consistentes, sem quadra faltando nem prédio duplicado): casas, prédios, fábricas, serviços, árvores, parques, postes, semáforos e placas; listas de LOD e materiais |
| `src/actors.rs` | Carros, caminhões, pedestres e trens de carga, com LOD próprio (rodas, faróis, luz de freio, ciclo de caminhada) |
| `src/custom.rs` | **Modelos importados do Blender**: vagas por tipo de objeto, variações, ângulo e cor das instâncias |
| `src/freight.rs` | Estação de carga, depósitos, divisão 70% trem / 30% caminhão, trens de carga |
| `src/math.rs` | Matrizes e câmera orbital (zoom, giro, inclinação que acompanha o zoom) |
| `src/rng.rs` | Aleatoriedade, hash e ruído fractal para gerar terreno |
| `src/lib.rs` | Funções exportadas para a página |
| `web/index.html` | Interface, shaders WebGL2, entrada de mouse/toque |
| `build_web.py` | Embute o `.wasm` na página |
| `examples/bench_milhao.rs` | Mede a simulação com 1 milhão de pessoas (`cargo run --release --example bench_milhao`) |
| `examples/diag_cidadaos.rs` | Mostra mês a mês emprego, saúde, crime, deslocamento e migração do bairro inicial |
| `examples/bench_rotas.rs` | Mede o custo das rotas (`cargo run --release --example bench_rotas`) |
| `docs/MODELOS_BLENDER.md` | **Como fazer, exportar e importar modelos do Blender** para prédios, árvores, carros, pessoas… |
| `ferramentas/` | Scripts para o Blender: exportar coleções para o jogo e gerar os modelos de exemplo |
| `modelos_exemplo/` | Modelos `.glb` feitos no Blender (casa, torre, loja, fábrica, carro, ônibus, pessoa, árvores) e a cena `.blend` |
| `web/modelos/` | `.glb` colocados aqui vão embutidos na página publicada |
| `examples/bench_transito.rs` | Mede quantos veículos e pedestres ficam nas ruas da metrópole e o custo por passo |
| `docs/MODELOS_3D.md` | **Guia para melhorar casas, prédios, carros e todos os objetos 3D** |
| `tests/simulacao.rs` | Testes: a cidade cresce; sem energia/água nada cresce; desempenho |
| `tests/desastres.rs` | Testes: faixas de densidade e valor aparecem; exército derrota monstro e invasores; terremoto e tornado destroem |
| `tests/estradas.rs` | Testes: construir, trocar o tipo e acesso aos lotes |
| `tests/vias.rs` | Testes: 10+ tipos com larguras diferentes, curvas, ninguém se sobrepõe, pedestres na calçada, trilho simples, salvar a rede |
| `examples/bench_rede.rs` | Mede o trânsito por faixa na metrópole (`cargo run --release --example bench_rede`) |
| `tests/salvar.rs` | Testes: salvar e carregar devolve a mesma cidade e ela continua evoluindo igual |
| `tests/cidadaos.rs` | Testes: todo morador tem casa, as ocupações batem, polícia prende, hospital trata, inspetor acha as pessoas |
| `tests/carga.rs` | Testes: depósito precisa de estação, 70% da carga vai de trem, o trem tem os vagões certos |
| `tests/modelos.rs` | Testes: modelos importados substituem prédios, árvores e carros; ângulo; limite de veículos |
| `tests/servicos.rs` | Testes: polícia reduz crime, bombeiro apaga incêndio, ônibus/metrô/trem levam passageiros |

### Como o mapa de 4 milhões de tiles cabe e roda rápido

- Cada tile ocupa 4 bytes (tipo, nível, parte, flags) + 4 bytes de camadas
  (trânsito, valor, poluição, insatisfação). Esses vetores **são** as texturas da
  GPU: o shader do chão desenha rios, zonas e camadas a partir delas, em qualquer
  zoom. As vias têm malha 3D própria, montada por bloco. Memória total: ~55 MB.
- A simulação nunca percorre o mapa inteiro por passo: zonas, ruas, serviços e
  trechos com trânsito vivem em listas próprias. Um passo leva ~0,1 ms.
- Prédios e árvores são instâncias agrupadas em blocos; só blocos alterados são
  refeitos e só os visíveis são desenhados.

## Cidadãos

Cada morador é uma pessoa de verdade na simulação, com cerca de 30 bytes:
casa, trabalho, loja preferida, dinheiro, idade, se tem carro, comida, saúde,
lazer, felicidade, último acontecimento e hora de entrar no trabalho. O nome
é gerado a partir do número da pessoa.

- **Um dia na vida:** procura emprego perto de casa (se houver vaga), recebe
  salário ou aposentadoria, paga o custo de vida, vai trabalhar, faz compras
  quando a comida acaba, passeia no parque quando falta lazer, fica doente
  (mais com poluição e idade) e vai à clínica ou ao hospital, se houver.
- **Transporte:** até 6 quadras vai a pé; se houver linha com parada perto de
  casa e do destino, pega ônibus, metrô ou trem (se não estiver lotado);
  senão vai de carro, se tiver, ou a pé. O tempo até o trabalho conta no humor.
- **Crime:** sem emprego, sem dinheiro e infeliz, a pessoa pode virar bandido
  e assaltar perto de casa. Com polícia por perto, pode ser presa por 30 dias
  e, ao sair, metade larga o crime.
- **Vida e morte:** as pessoas envelhecem, se aposentam aos 65, têm filhos,
  morrem de velhice ou de doença sem tratamento e vão embora se ficarem muito
  infelizes. Gente de fora chega para ocupar as vagas quando há procura.
- **A cidade vem das pessoas:** a população é quem mora de verdade; lojas e
  fábricas precisam de funcionários para crescer; impostos saem de quem mora e
  trabalha; o desemprego puxa a abertura de novos empregos; crimes cometidos
  sobem o índice de crime do bairro.
- **Nas ruas:** uma amostra das viagens aparece como carros e pedestres. Cada
  um carrega a pessoa de verdade que está viajando.

**Inspetor (lupa, tecla I):** clique numa pessoa ou num carro para ver a
ficha: nome, idade, situação, onde está agora, onde mora, trabalha e compra,
como se desloca e quanto tempo leva, dinheiro, comida, saúde, lazer,
felicidade e o que aconteceu com ela por último. A casa fica em verde, o
trabalho em azul e o caminho entre eles em amarelo. "Seguir com a câmera"
acompanha a pessoa pela cidade. Clique num prédio para ver quem mora ou
trabalha ali.

**Metrópole de teste:** o botão "Criar metrópole de teste (1 milhão)"
constrói uma cidade densa com 1 milhão de moradores, delegacias, bombeiros,
hospitais e 6 linhas de metrô. Nela, os prédios ficam fixos e energia e água
são ilimitadas (modo de teste), mas as pessoas vivem normalmente.

### Desempenho com 1 milhão de pessoas

Medido no WebAssembly, fora do navegador (Node):

| O quê | Tempo |
|---|---|
| Um dia de vida de cada pessoa | ~340 ns por pessoa (1 milhão ≈ 0,33 s por dia de jogo) |
| Jogo a 1× (20 passos/s) | ~5,4 ms por quadro em média, e as pessoas acompanham o ritmo |
| Memória total | ~116 MB (30 MB são as pessoas) |
| Salvar 1 milhão de pessoas | ~2 s no navegador, **3,7 MB** comprimido |

A página dá às pessoas um orçamento de ~5 ms por quadro e ajusta sozinha
quantas processa. Em aparelhos lentos, ou nas velocidades 2× e 4×, o "dia" de
cada pessoa demora mais para chegar, mas ninguém perde dinheiro nem idade: o
próximo passo compensa os dias atrasados.

A simulação roda no mesmo processo da tela, em fatias. Colocar num processo
separado (Web Worker) exigiria mover também a montagem do 3D e copiar o mapa
de 16 MB entre os dois a cada mudança. Com as medições acima, as fatias bastam
na velocidade 1×. O worker fica como próximo passo se as velocidades 2× e 4×
com 1 milhão precisarem ficar em dia.

## Gráficos

- **LOD por bloco:** perto da câmera, modelo completo; na média, corpo,
  telhado e janelas; longe, uma caixa por lote. Os limites (`LOD_NEAR`,
  `LOD_FAR`, `LOD_TREES`) ficam no começo do script de `web/index.html`, e
  `window.cidadeLod` mostra quantos blocos estão em cada nível.
- **Janelas, luzes e semáforos** são materiais do shader: o canal alfa da
  cor de cada instância diz qual material usar. Não custam geometria.
- **Dia e noite:** o dia dura 8 minutos. O botão ao lado da velocidade
  (ou a tecla N, ou um clique na hora) desliga a noite, e aí o sol só anda
  entre 7h e 17h. A escolha fica gravada. É só visual e não muda a
  simulação.
- **Modelos do Blender:** qualquer objeto pode usar um modelo `.glb` feito
  no Blender. Importe pelo cartão Modelos 3D ou arraste para o mapa. Veja
  [docs/MODELOS_BLENDER.md](docs/MODELOS_BLENDER.md).
- **Trânsito:** até 200 mil veículos e pedestres nas ruas numa metrópole.
  O limite pode ser baixado em Transporte → Máximo nas ruas.
- **Chão:** zebras, setas, asfalto gasto, luz dos postes e sombra falsa dos
  prédios saem do shader `GROUND_FS`, sem instâncias.

## Carga ferroviária

Construa uma **estação de carga** encostada num trilho e **depósitos**
encostados nela. Enquanto houver capacidade, 70% dos caminhões das
indústrias num raio de 45 quadras vão só até a estação. Cada estação
aguenta 400 viagens por mês, mais 300 por depósito. Um trem de carga leva
a mercadoria até a ponta da ferrovia, e cada depósito aumenta o trem em 2
vagões. A estação também recebe mercadoria de fora, que sai de caminhão
para as lojas. Indústrias atendidas podem crescer uma faixa a mais.

## Salvamento

- A cidade é salva **automaticamente a cada 2 minutos** e também quando você
  troca de aba ou fecha a página. Ao abrir o jogo de novo, a última cidade volta
  sozinha.
- O salvamento fica no próprio navegador (IndexedDB), comprimido. O terreno
  não é salvo, só a semente e as quadras que mudaram. As pessoas são salvas
  uma a uma: o bairro inicial ocupa ~100 KB e 1 milhão de moradores, ~3,7 MB.
- **Exportar arquivo** baixa um `.cidade` para guardar ou levar para outro
  computador; **Importar arquivo** abre um desses arquivos.
- Cada navegador (e cada endereço onde o jogo está publicado) tem o seu próprio
  salvamento. Limpar os dados do site apaga a cidade salva, então exporte uma
  cópia de vez em quando.
- "Novo mapa vazio" pede um segundo clique para confirmar, porque substitui a
  cidade salva.

## Interface

- **Barras de ícones:** ferramentas, camadas do mapa e velocidade aparecem só
  como ícones. O ícone se anima quando o cursor passa por cima ou quando a
  ferramenta está ativa, e uma dica mostra o nome, o custo e o atalho. No
  celular, a dica aparece ao tocar. A ferramenta atual fica escrita no canto do mapa.
- **Gráficos:** logo abaixo do mapa, com abas (População, Empregos, Finanças,
  Demanda, Trânsito, Transporte público, Energia e água, Segurança e saúde,
  Prédios). Escolha o período (3 meses, 1 ano, 2 anos), passe o cursor para ver
  os valores de cada dia, ou ligue **Tabela** para ver os números. As setas ←/→
  trocam de aba quando o foco está nelas.

## Vias e trânsito

As vias formam uma **rede livre**: cada trecho é uma curva entre dois nós, e as
vias podem fazer curvas, se cruzar em qualquer ângulo e ter larguras
diferentes. Zonas e prédios continuam na grade de quadras (1 quadra = 10 m):
a via ocupa as quadras por onde passa, conforme a largura.

| Via | Faixas | Largura | Velocidade | R$ por quadra | Observação |
|---|---|---|---|---|---|
| Beco | 1 + 1 | 5,8 m | muito lenta | 6 | asfalto gasto, calçada estreita |
| Rua de barro | 1 + 1 | 7,2 m | lenta | 4 | sem calçada (pedestres no acostamento) |
| Rua de mão única | 2 → | 9,0 m | normal | 10 | clique de novo com a mesma ferramenta para inverter o sentido |
| Rua de mão dupla | 1 + 1 | 10 m | normal | 10 | faixa amarela no meio |
| Rua de concreto | 1 + 1 | 11 m | normal + | 14 | placas de concreto, calçada larga |
| Avenida 2 + 2 | 2 + 2 | 17 m | rápida | 25 | canteiro central |
| Avenida 3 + 3 | 3 + 3 | 26 m | rápida | 40 | canteiro com árvores |
| Avenida 4 + 4 | 4 + 4 | 30 m | rápida | 55 | |
| Autoestrada 3 + 3 | 3 + 3 | 28 m | muito rápida | 70 | sem pedestres nem acesso aos lotes, guard-rail |
| Autoestrada de via única | 1 → | 7 m | muito rápida | 35 | rampas e alças |
| Trilho simples | 1 linha | 4 m | — | 25 | um trem por vez, nos dois sentidos |
| Trilho duplo | 1 + 1 | 8 m | — | 40 | uma linha por sentido |

- **Construir:** arraste do começo ao fim (reta) ou use **Curva**: três
  cliques (começo, ponto que puxa a curva, fim); a curva seguinte começa onde a
  anterior terminou. Cruzar outra via cria o cruzamento sozinho; a ponta
  encosta em vias e nós próximos. Arrastar sobre uma via existente troca o tipo
  dela (pagando a diferença).
- **Cada veículo anda numa faixa** do seu lado da via e muda de faixa nos
  cruzamentos conforme a rota. Os **pedestres andam só na calçada**.
- **Conflito:** cada faixa é uma fila; ninguém ocupa o espaço de outro. Quem
  vem atrás freia e espera; no cruzamento sem semáforo entra um braço por vez,
  e ninguém entra se não couber do outro lado (para não travar o cruzamento).
  Cruzamentos de avenidas, ou com 4 braços, ganham semáforo.
- **Trilhos** usam a mesma rede e as mesmas regras: curvas, filas, passagem
  de nível. No trilho simples o trem só entra se não vier outro no sentido
  contrário.
- Veículos presos há muito tempo desistem da viagem (contam como atraso).
- A capacidade agora é física: quantos carros cabem depende do comprimento e
  das faixas das vias. Numa metrópole de 1 milhão feita só de ruas de mão
  dupla, ~30 a 40 mil veículos e pedestres ocupam as ruas ao mesmo tempo e o
  movimento custa ~4 ms por passo; avenidas aumentam a capacidade.
- **Cidades salvas antes desta versão** são convertidas: cada sequência de
  ruas vira trechos retos do tipo equivalente (terra → barro, rua → mão dupla,
  avenida → 2 + 2, via expressa → autoestrada 3 + 3). Avenidas e autoestradas
  ficam mais largas que antes e podem encostar em prédios vizinhos.
- O metrô continua em túnel, com rotas próprias.

## Regras

- **Vias** (veja a seção [Vias e trânsito](#vias-e-trânsito)): 10 tipos de
  via e 2 de trilho, retas ou em curva, cada um com a sua largura.

- A cidade só cresce ligada às **rodovias estaduais** que cruzam o mapa.
- **Energia e água**: cada rede de ruas conectadas é também a rede de cabos e
  canos. Usinas e bombas abastecem a rede em que estão ligadas. Se a produção não
  cobre o consumo, parte dos lotes fica sem, para de crescer e é abandonada.
  - Usina a carvão (3×3): 500 MW, polui muito.
  - Turbina eólica (1×1): 70 MW, limpa.
  - Bomba d'água (2×2): 6.000 m³/h, precisa encostar num rio ou lago.
  - Caixa d'água (1×1): 1.000 m³/h, em qualquer lugar.
- **Demanda RCI**: moradores seguem empregos; comércio e indústria seguem moradores.
- **Trânsito**: viagens reais (casa → trabalho, indústria → comércio, visitantes).
  Atrasos repetidos deixam o lote insatisfeito e ele encolhe.
- Indústria e usinas poluem; parques, água e árvores valorizam o terreno.
- **Polícia** (delegacia 2×2): reduz o crime num raio de 26 quadras e patrulha
  os pontos mais perigosos. Crime alto derruba o valor, trava o crescimento e
  expulsa moradores.
- **Bombeiros** (quartel 2×2): prédios pegam fogo de vez em quando (indústrias
  mais). O quartel mais próximo manda um caminhão pela rede viária; se ele não
  chega antes do fogo acabar, o prédio vira ruína. Sem cobertura, o fogo se espalha.
- **Saúde** (clínica 1×1, hospital 3×3): a saúde dos moradores cai com a
  poluição e sobe com atendimento. Saúde baixa impede o crescimento e, muito
  baixa, faz as pessoas irem embora. Ambulâncias circulam pelos bairros.
- **Transporte público**: escolha a ferramenta de linha e clique nas paradas em
  ordem; depois "Concluir linha" (ou Enter). A linha roda em círculo.
  - Ônibus: paradas em ruas; andam no trânsito junto com os carros.
  - Metrô: clique em terreno livre para abrir estações; corre em túnel reto,
    sem trânsito (aparece na camada Transporte).
  - Trem: construa trilhos (retos ou em curva; cruzam ruas em passagem de
    nível) e estações encostadas neles; o trem segue os trilhos. No trilho
    simples só passa um trem por vez.
  - Quem mora perto de uma parada e vai para perto de outra da mesma linha pode
    deixar o carro em casa. Linhas lotadas perdem passageiros para o carro.
    Passagens rendem R$ 2 por viagem; cada veículo tem custo mensal.

- **Densidade (6 faixas)**: casa → sobrado → prédio baixo → prédio médio →
  torre → arranha-céu (comércio e indústria têm equivalentes). Faixa 2 pede
  valor do terreno; 3, pouca poluição; 4, polícia e saúde; 5, transporte
  público a pé; 6, metrô ou trem a pé (indústria: bombeiros, transporte, trem).
- **Valor financeiro (6 faixas)**: baixa renda, popular, classe média, média
  alta, alto padrão e luxo. Segue o valor do terreno e cai com poluição e
  crime. Luxo paga 5× mais imposto que baixa renda; pobreza aumenta o crime.
  Visual muda: barracos com caixa d'água na laje, casas com piscina, torres de
  vidro com coroa dourada; indústria de alto valor vira polo de tecnologia.
- **Desastres e defesa** (painel na lateral, ou aleatórios se ligados):
  - Monstro gigante: vem de longe, pisa e destrói, cospe fogo. A **base
    militar** (3×3) manda tanques pelas ruas e helicópteros pelo ar; derrotá-lo
    rende R$ 8.000. Sem exército, ele destrói bairros e vai embora.
  - Invasão: comboio de tanques entra pela rodovia e ataca prédios; exército e
    delegacias revidam.
  - Terremoto: epicentro perto do centro, a câmera treme, prédios perdem
    andares ou desabam, incêndios, ruas se rompem (a rodovia resiste).
  - Tornado: funil que cruza a cidade destruindo o que toca.
  - Escombros ficam alguns dias no lugar e depois o lote pode ser reconstruído.


