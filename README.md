# Thiago TV OS

Tela inicial personalizada para a minha Samsung Smart TV (UN65TU8000, 65", Tizen 5.5), sem anúncios, com os apps de streaming que eu uso, clima de São Paulo e sugestões de filmes e séries.

Este projeto é um fork de [simbasjungles/free-my-tv](https://github.com/simbasjungles/free-my-tv) (licença MIT). Todo o crédito do app vai para o autor original. Aqui eu documento e mantenho a minha instalação.

## O que é

Um app web empacotado como `.wgt` (formato de apps Tizen) que roda na TV como uma tela inicial alternativa. Ele **não** faz root nem altera o firmware: a tela inicial original da Samsung continua instalada, e o app pode ser removido a qualquer momento.

O que ele mostra:

- Prateleira com os apps de streaming da TV, sem propaganda.
- Sugestões de filmes e séries em alta (fonte: TMDB, região BR), apenas dos apps que a TV tem.
- Clima hora a hora e horário do nascer e pôr do sol de São Paulo.
- Preços de criptomoedas em BRL (Binance).
- Tema claro de dia e escuro à noite, seguindo o pôr do sol.
- Proteção de tela com fotos das sugestões e relógio.

## O que foi feito

1. Criado este repositório com o GitHub CLI (`gh repo create`), público, a partir do código do free-my-tv.
2. Localizada a TV na rede local (`192.168.15.6`) pela API da Samsung (porta 8001) e conferido o modelo.
3. Ativado o Developer Mode na TV (passos abaixo).
4. Criado o `config.properties` com as minhas preferências (cidade, região, unidades, chave TMDB).
5. Instaladas as ferramentas de assinatura da Samsung (`tizen` CLI) a partir do repositório oficial, com checksums SHA256 conferidos.
6. Gerado um certificado de autor Tizen (`thiagotv`) e um perfil de segurança.
7. Build assinado: `python3 tizen/build.py --sign thiagotv`, que gera `FreeMyTV-tizen.wgt`.
8. Instalação pela rede: `python3 tizen/install.py 192.168.15.6`, que fala SDB (porta 26101) direto com a TV.

## Configuração da TV (Samsung, Tizen)

Funciona em TVs Samsung de 2017 em diante (Tizen 3+).

1. Na TV, abra **Apps**.
2. Digite `1 2 3 4 5`. Se o controle não tem teclado numérico, use o botão **123** no controle ou a aba de números do controle virtual no app SmartThings.
3. Ligue o **Developer mode**.
4. Em **Host PC IP**, digite o IP do computador que vai fazer a instalação (o meu Mac: `192.168.15.5`).
5. Reinicie a TV.

Opcional: ative **Autorun Last App** nas configurações da TV (Geral, Smart Features; o lugar varia por modelo) para a TV ligar direto no app.

## Como reproduzir

Requisitos: macOS ou Linux, Python 3, JDK 17 e as ferramentas `tizen` da Samsung.

```bash
# 1. configuração local (fica fora do git)
cp config.example.properties config.properties
# edite: tmdb_key, tmdb_region, latitude, longitude, city, currency, temperature, language

# 2. certificado de assinatura (uma vez; a senha é sua, digite só no seu terminal)
tizen certificate -a thiagotv -n "Seu Nome" -p <senha> -f thiagotv
tizen security-profiles add -n thiagotv -a ~/tizen-studio-data/keystore/author/thiagotv.p12 -p <senha>

# 3. build assinado
python3 tizen/build.py --sign thiagotv

# 4. instalar pela rede
python3 tizen/install.py <IP_DA_TV>
```

Detalhes de como obter o CLI `tizen` sem o Tizen Studio (útil em Mac com Apple Silicon) estão em [`CLAUDE.md`](CLAUDE.md), seção "Installing it on a Samsung TV (Tizen)".

### Minhas configurações

| Chave | Valor |
|---|---|
| `tmdb_region` | `BR` |
| `latitude` / `longitude` | `-23.5505` / `-46.6333` |
| `city` | São Paulo |
| `language` | `pt` |
| `temperature` | `C` |
| `currency` | `BRL` |

A chave do TMDB (`tmdb_key`) é pessoal e **nunca** vai para o git. Para criar uma: [themoviedb.org](https://www.themoviedb.org) → Settings → API.

## Arquivos que ficam fora do git

`config.properties` (contém a chave TMDB), `*.wgt` (build assinado) e os certificados em `~/tizen-studio-data`. **Guarde o certificado e a senha:** atualizações do app precisam ser assinadas com o mesmo certificado, senão a TV recusa.

## Atualizar ou remover

- **Atualizar:** altere o código ou o `config.properties`, rode o build e o `install.py` de novo.
- **Remover:** `python3 tizen/install.py <IP_DA_TV> --uninstall`, ou na TV: Apps, Configurações, Free My TV, Excluir.

## Estrutura

- `web/`: o app (HTML, CSS, JS compatível com navegadores antigos de TV).
- `tizen/`: empacotamento (`build.py`), instalação via SDB (`install.py`) e `config.xml`.
- `webos/`, `app/`: versões para LG e Google TV, vindas do projeto original, sem uso aqui.
- `tools/`: verificações (`tools/check.sh`).

## Sincronizar com o projeto original

```bash
git fetch upstream
git merge upstream/main
```

## Licença

MIT, a mesma do projeto original. Veja [`LICENSE`](LICENSE).
