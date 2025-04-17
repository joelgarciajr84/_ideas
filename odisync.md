 História 1: Comparação de arquivos JSON entre dias consecutivos usando AWS Glue
Narrativa / Visão do usuário
Como engenheiro de dados, quero um Glue Job que compare arquivos JSON dos dias atual e anterior em dois buckets, para identificar diferenças e salvá-las em um bucket específico de "diffs".

DOR
Buckets de entrada e saída definidos.

Glue habilitado na conta.

Estrutura dos arquivos JSON acordada.

Descrição
Um job do AWS Glue será executado diariamente, lendo arquivos JSON de dois buckets (dia atual e dia anterior), comparando seus conteúdos e salvando as diferenças encontradas em um terceiro bucket, o bucket_diffs.

Premissas de desenvolvimento
Os arquivos possuem uma chave única que permite a comparação.

A estrutura dos arquivos permanece consistente.

A data do processamento será passada como parâmetro (ex: "hoje").

Informações técnicas
Linguagem: PySpark no Glue.

Buckets:

bucket_ontem/ano/mes/dia/

bucket_hoje/ano/mes/dia/

bucket_diffs/ano/mes/dia/

Output: arquivos JSON com diferenças.

IAM Role do Glue com permissões de leitura e escrita nos 3 buckets.

Passo a passo
Criar buckets S3 (ontem, hoje, diffs).

Criar IAM Role com permissões nos buckets.

Desenvolver script PySpark de comparação.

Criar Glue Job com parâmetros de data.

Criar trigger diária via Glue ou EventBridge.

Critério de aceite (DOD)
Glue Job criado e visível no console.

Arquivos de diffs gerados corretamente.

Job executa com parâmetros de data e salva saída no bucket correto.

Permissões validadas com sucesso.

🔹 História 2: Disparo de eventos SQS a partir de arquivos de diff salvos no S3
Narrativa / Visão do usuário
Como engenheiro de plataforma, quero que cada novo arquivo de diffs salvo no bucket envie uma mensagem a uma fila SQS, para que outras aplicações possam reagir a essas mudanças.

DOR
Bucket de diffs já criado.

Permissões de eventos S3 para SQS disponíveis.

Descrição
A cada novo arquivo JSON salvo no bucket de diffs, uma mensagem é enviada automaticamente para uma fila SQS com metadados básicos (nome do arquivo, data, etc).

Premissas de desenvolvimento
Os eventos serão apenas de "PUT".

A fila será do tipo standard.

Mensagens terão formato simples e padronizado.

Informações técnicas
Fila SQS: diffs-events-queue

Bucket: bucket_diffs

Notificação: evento PUT → envia mensagem com caminho do arquivo.

IAM policy para permitir s3:PutObject → sqs:SendMessage.

Passo a passo
Criar fila SQS com policy de acesso S3.

Adicionar notificação no bucket_diffs.

Validar envio automático ao subir arquivos de teste.

Critério de aceite (DOD)
Arquivos novos no bucket geram mensagens na fila.

Mensagens seguem padrão { path: "...", date: "..." }.

Fila visível e monitorável no console AWS.

IAM policies auditadas e seguras.

🔹 História 3: ECS Task que consome fila SQS e envia eventos para Kafka
Narrativa / Visão do usuário
Como engenheiro de dados, quero um ECS agendado que consome eventos da fila SQS de diffs e envie os dados para um tópico Kafka, para alimentar fluxos downstream.

DOR
Fila SQS configurada.

Kafka acessível (público ou via VPC).

Definição do payload Kafka.

Descrição
Uma task do ECS será executada diariamente (ou sob demanda) para ler mensagens da fila SQS, processar os arquivos referenciados no S3, extrair dados e publicar em um tópico Kafka.

Premissas de desenvolvimento
Cada mensagem na fila contém o caminho de um arquivo válido no bucket de diffs.

O ECS possui permissão para acessar o S3 e Kafka.

O formato da mensagem Kafka já é conhecido.

Informações técnicas
Serviço: ECS Fargate com task agendada via EventBridge.

Docker container custom com código Python (ou outro) para:

Ler mensagens da fila SQS.

Baixar e processar os arquivos no S3.

Enviar o conteúdo para o tópico Kafka.

Kafka configurável por variáveis de ambiente ou Secrets Manager.

Logs via CloudWatch.

Passo a passo
Criar container com app consumidor da SQS e publisher Kafka.

Criar IAM Role com acesso a S3, SQS, Kafka.

Criar cluster ECS e task definition.

Criar agendamento diário às 2h via EventBridge.

Configurar CloudWatch Logs para monitoramento.

Critério de aceite (DOD)
Task ECS executa com sucesso e sem erros.

Mensagens da fila SQS são processadas e arquivos baixados.

Eventos são publicados no tópico Kafka com sucesso.

Logs visíveis no CloudWatch.

Task pode ser monitorada e reagendada facilmente.

