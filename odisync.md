Épico: Comparação e envio de diffs entre arquivos JSON de buckets S3
🧪 História 1: Como engenheiro de dados, quero um Glue Job que compare arquivos JSON de dois buckets (ontem e hoje), para identificar e salvar as diferenças entre eles.
Critérios de Aceitação:

O Glue Job deve ler arquivos JSON dos buckets bucket_ontem/ e bucket_hoje/ (parametrizáveis).

A comparação deve ocorrer por chave única presente nos arquivos.

As diferenças devem ser salvas como arquivos JSON no bucket bucket_diffs/YYYY-MM-DD/.

O job deve rodar de forma parametrizada com a data do dia atual (ex: via trigger, step function, etc).

Tarefas Técnicas:

 Criar script PySpark para comparar os dados.

 Criar Glue Job com IAM Role apropriada.

 Configurar trigger/evento para execução diária (pode ser por agendamento ou manual).

📤 História 2: Como engenheiro de plataforma, quero que cada novo arquivo salvo no bucket de diffs envie uma mensagem a uma fila SQS, para processamentos posteriores.
Critérios de Aceitação:

Cada novo arquivo no bucket_diffs/YYYY-MM-DD/ deve gerar uma mensagem na fila SQS.

A mensagem deve conter no mínimo o path do arquivo salvo e a data.

O bucket de diffs deve ter notificação configurada para a fila SQS.

A fila deve ser criada com políticas apropriadas de acesso.

Tarefas Técnicas:

 Criar bucket de diffs com configuração de evento S3 → SQS.

 Criar fila SQS com permissões adequadas.

 Configurar notificação de evento no bucket.

⏰ História 3: Como engenheiro de dados, quero um ECS Task agendado diariamente às 2h que leia os arquivos de diffs do dia atual e envie os eventos para um tópico Kafka.
Critérios de Aceitação:

ECS Task deve ser agendada via EventBridge para execução diária às 2h da manhã.

O container deve ler os arquivos do path bucket_diffs/YYYY-MM-DD/.

Cada arquivo deve ser lido e seu conteúdo publicado em um tópico Kafka.

O tópico Kafka e credenciais de acesso devem ser configuráveis.

Tarefas Técnicas:

 Criar imagem Docker que lê do S3 e envia para Kafka.

 Criar cluster ECS (ou usar existente).

 Criar ECS Task Definition e EventBridge Rule.

 Garantir que o task tenha acesso ao S3 e Kafka.

