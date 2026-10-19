# Oracle VM Always Free retry

Este repositório tenta criar uma única VM Ubuntu ARM na Oracle Cloud quando houver capacidade. O fluxo é automático a cada 30 minutos e também pode ser iniciado manualmente em **Actions → Criar VM Oracle Always Free quando houver capacidade → Run workflow**.

## Limites de segurança do fluxo

- Usa somente `VM.Standard.A1.Flex`, com **1 OCPU e 6 GB de memória**.
- Só executa na **região home** da tenancy.
- Antes de criar, consulta as instâncias A1 e os volumes em todos os compartimentos acessíveis. Para e falha se a nova VM exceder o limite Always Free do Oracle para tenancy Always Free: **2 OCPUs, 12 GB de memória ou 200 GB de armazenamento combinado**.
- Usa um volume de inicialização de **47 GB**, sem volumes extras e sem backup automático configurado.
- Não atribui IPv4 público e não tenta formas, imagens licenciadas, volumes ou serviços pagos como alternativa.
- Se não houver capacidade em um domínio, tenta os outros domínios da região home. Se todos estiverem lotados, o próximo ciclo tenta novamente.
- Se já existir uma VM chamada `oracle-vm-retry`, não cria outra.

As verificações são conservadoras: se uma consulta de recursos não funcionar ou não puder conferir os limites, o fluxo para sem criar.

## Antes de ativar

A conta Oracle precisa ter uma chave de API configurada para OCI CLI. **Nunca coloque a chave privada, o arquivo de configuração ou qualquer credencial no código, em Issues ou neste README.** Cadastre a chave privada somente em **Settings → Secrets and variables → Actions → New repository secret**.

### Secrets do repositório

- `OCI_CONFIG_CONTENT`: conteúdo do arquivo `~/.oci/config`, com um perfil `[DEFAULT]`. No campo `key_file`, use o caminho do runner `~/.oci/oci_api_key.pem`.
- `OCI_PRIVATE_KEY_PEM`: conteúdo completo da chave privada PEM correspondente à chave pública cadastrada no usuário OCI.

### Variables do repositório

Cadastre em **Settings → Secrets and variables → Actions → Variables**:

- `OCI_TENANCY_OCID`: OCID da tenancy.
- `OCI_COMPARTMENT_OCID`: compartimento onde a VM será criada.
- `OCI_SUBNET_OCID`: subnet existente na região home.
- `OCI_REGION`: nome da região home (por exemplo, `sa-saopaulo-1`, somente se for de fato a região home).
- `OCI_UBUNTU_ARM_IMAGE_OCID`: OCID de uma imagem Ubuntu ARM64 marcada pela Oracle como Always Free Eligible na região home.
- `OCI_SSH_PUBLIC_KEY`: chave pública SSH correspondente à sua chave privada SSH. Ela não é a chave privada da API.

Use a configuração OCI gerada pelo OCI CLI. O perfil deve ter `tenancy`, `user`, `fingerprint`, `key_file` e `region`. O fluxo cria a chave PEM de API no runner a partir do Secret, mantendo o arquivo fora do repositório.

## Importante

A Oracle documenta o A1 como gratuito dentro das cotas Always Free e informa que os limites de uma tenancy Always Free são menores que os limites de uma conta atualizada. Este fluxo **não atualiza a conta**. Ele foi configurado para não solicitar recursos adicionais acima dos limites verificados. Ainda assim, nenhum script pode garantir cobrança zero se a conta, a região, a classificação de elegibilidade de um recurso ou as regras da Oracle mudarem; confira sempre no Console se a forma e a imagem estão marcadas **Always Free Eligible** e não escolha **Upgrade**.

O repositório é público. O código e os nomes das variáveis são visíveis, mas os valores cadastrados como GitHub Actions Secrets não são publicados pelo GitHub. Não execute workflows alterados por terceiros antes de revisar o código.
