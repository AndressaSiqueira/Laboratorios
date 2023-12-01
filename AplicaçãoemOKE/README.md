## Pré-Requisitos
- Todos os serviços precisam ter suas politicas de acesso regulares para que não tenha problemas relacionados á acesso. Ex: grupo dinamico que permite que OCI Registry e OKE se comuniquem.
- A aplicação precisa ter uma rota de exposição para que o serviço seja comunicável, no caso de uma aplicação python, poderiamos adicionar a seguinte linha:
```bash
@app.route('/answer', methods=['POST'])
def get_answer():
    data = request.get_json()
    question = data['pergunta']
    result = rag_pipeline.run(query=question)
    resultado_final = print_answer_2(result)
    return jsonify({'answer': resultado_final})
    
if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=3000
```
Nesse caso estamos usando o Flask, onde há uma rota de POST aceitável para o path /answer que faz determinada ação, rodando no ambiente local para a porta 3000.

## Provisionando Cluster OKE
Efetuar provisionamento do cluster OKE conforme [documentação oficial da Oracle](https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengcreatingclusterusingoke_topic-Using_the_Console_to_create_a_Quick_Cluster_with_Default_Settings.htm#create-quick-cluster)

## Criando imagem docker

Esse passo pode ser feito inicialmente no cloud shell ou em um máquina virtual dedicada, em um processo futuro também pode ser automatizado via esteira Devops.

1. Criar a pasta do projeto com os arquivos necessários para a construção da imagem:

- Dockerfile: um arquivo de configuração usado no Docker, uma plataforma de contêineres que permite empacotar, distribuir e executar aplicativos em contêineres. Exemplo prático  [aqui](https://github.com/AndressaSiqueira/Laboratorios/blob/main/Aplica%C3%A7%C3%A3oemOKE/Dockerfile.txt)
- App.py : Arquivo da lógica da aplicação.
- Requirements.txt: Todas as dependências necessárias para aquela aplicação, vai depender da linguagem, ex: nodejs teriamos um arquivo package.json.

2. Dentro da pasta do projeto efetuar os seguintes comandos:
```bash
docker login <region-key>.ocir.io
```
Código da sua região. [Lista para ver a sigla da sua região](https://docs.oracle.com/pt-br/iaas/Content/General/Concepts/regions.htm) Ex: Brasil Vinhedo=vcp.ocir.io
Efetuar login com username (Opção que aparece com nome da tenancy/usuário) e senha, a senha será um auth token que você pode gerar ou tenha acesso no seu usuário.[Como gerar auth token](https://docs.oracle.com/en-us/iaas/Content/Registry/Tasks/registrygettingauthtoken.htm)

```bash
docker build -t <region-key>.ocir.io/<tenancy-namespace>/repositorio/nome_da_imagem .
```
Ao final da build podemos fazer o push para o OCIR

```bash
docker push <region-key>.ocir.io/<tenancy-namespace>/repositorio/nome_da_imagem
```

Onde:
region-key: Código da sua região. [Lista para ver a sigla da sua região](https://docs.oracle.com/pt-br/iaas/Content/General/Concepts/regions.htm) Ex: Brasil Vinhedo=vcp.ocir.io
tenancy-namespace: ID único de cada tenancy que pode ser encontrado em Profile > Tenancy > Object storage namespace:
Repo-name : Nome do repositório Oracle Registry que você quer mandar essa imagem
tag: tag utilizada para diferenciação de versões, podendo ser latest ou v1/v2 e etc.

Verfique se sua imagem encontra-se no OCI Registry.

## Deploy da Aplicação no Kubernetes

1. Acesse o OKE 
2. Crie um arquivo chamado Nome_do_arquivo.yaml , esse será o arquivo de deployment da nossa aplicação. 
O conteudo deve ser como [esse arquivo](https://github.com/AndressaSiqueira/Laboratorios/blob/main/Aplica%C3%A7%C3%A3oemOKE/manifesto_oke.yaml)

3. Dê os comandos:
```bash
kubectl create secret docker-registry ocisecret --docker-server=<region-key>.ocir.io --docker-username='<tenancy-namespace>/<oci-username>' --docker-password='<oci-auth-token>' --docker-email='<email-address>'
```
```bash
kubectl apply -f  Nome_do_arquivo.yaml
```
```bash
kubectl get pods -A 
```
(Para verificar se os pods da aplicação estão em status 'running')
```bash
kubectl get svc -A 
```
(Para pegar o IP External gerado, esse é o IP e porta da nossa aplicação)
