# Moodle AWS Infrastructure

### Infraestrutura como código do Moodle usando Amazon CloudFormation

Ver no [GitHub](https://github.com/vitor-msp/moodle-infra).

#### Importação de certificado TLS

1. Gere um certificado TLS e abra a console da AWS no AWS Certificate Manager.

![import certificate](images/1-import-certificate.png)

2. Copie o conteúdo do certificado nos campos apropriados e salve.

![import certificate](images/2-import-certificate.png)

3. Copie o ARN do certificado.

![import certificate](images/3-import-certificate.png)

#### Criação da stack

4. Acesse o Amazon CloudFormation para criar uma nova stack.

![create stack](images/4-create-stack.png)

5. Faça o upload do arquivo [moodle-infra-aws.yml](moodle-infra-aws.yml).

![create stack](images/5-create-stack.png)

6. Preencha todos os parâmetros exigidos e apenas os vazios (sem valor default). Busque o AMI id do Ubuntu 24.04 LTS na região da AWS em que está e crie a stack. Pode levar até uns 20 minutos para finalizar a criação. Quando finalizar, os outputs estarão disponíveis, os quais serão usados em seguida.

![create stack](images/6-create-stack.png)

7. Crie outra stack, desta vez utilizando o arquivo [generate-moodle-ami-aws.yml](generate-moodle-ami-aws.yml), preencha todos os parâmetros exigidos e novamente use o AMI id do Ubuntu 24.04 LTS na região da AWS em que está. A maioria dos parâmetros será algum output da stack anterior. Esta stack provavelmente finalizará em menos de 3 minutos.

![create stack](images/7-create-stack.png)

8. Copie o output SSHCommand da stack anterior e execute o comando na pasta onde seu key pair está contido para acessar a máquina de configuração. Em seguida, execute o comando `tail -f /tmp/moodle-installation.log` e aguarde até que seja apresentada uma mensagem de instalação finalizada, como na imagem abaixo. Este processo pode levar até uns 20 minutos.

![create stack](images/8-create-stack.png)

#### Configuração do Moodle

9. Acesse a URL do output EC2Url da stack anterior e prossiga mesmo com a notificação de conexão insegura. Esta mensagem é exibida pois o CA do certificado provavelmente não é reconhecido.

![configure moodle](images/9-configure-moodle.png)

10. Selecione o idioma que preferir.

![configure moodle](images/10-configure-moodle.png)

11. Prossiga sem alterações. O diretório de dados deve ser /var/www/moodledata.

![configure moodle](images/11-configure-moodle.png)

12. Prossiga sem alterações. O driver deve ser PostgreSQL.

![configure moodle](images/12-configure-moodle.png)

13. Preencha as informações de conexão com o banco de dados.

![configure moodle](images/13-configure-moodle.png)

14. Leia as notas e confirme.

![configure moodle](images/14-configure-moodle.png)

15. Nesta tela, todos os itens devem estar marcados com ok. Apenas siga em frente.

![configure moodle](images/15-configure-moodle.png)

16. Preencha o que for obrigatório e o que achar necessário.

![configure moodle](images/16-configure-moodle.png)

17. Preencha o que for obrigatório e o que achar necessário.

![configure moodle](images/17-configure-moodle.png)

18.  Preencha um e-mail de suporte.

![configure moodle](images/18-configure-moodle.png)

19. Se for direcionado à tela inicial do Moodle, sua instalação foi finalizada com sucesso.

![configure moodle](images/19-configure-moodle.png)

#### Configuração do cache

20. Acesse "administração do site" -> "plugins".

![configure cache](images/20-configure-cache.png)

21. Em "caching", acesse "configuração".

![configure cache](images/21-configure-cache.png)

22. Logo no início da página, em "redis", acesse "adicionar instância".

![configure cache](images/22-configure-cache.png)

23. Preencha as informações de acesso ao cache e salve. No campo "servidores" use o hostname e a porta separados por dois pontos, como na imagem. Marque o checkbox "use a criptografia tls". Você será direcionado a página anterior.

![configure cache](images/23-configure-cache.png)

24. No final da página, na seção "armazenamentos utilizados quando nenhum mapeamento está presente", acesse "editar mapeamento".

![configure cache](images/24-configure-cache.png)

25. Preencha "aplicação" e "sessão" com a instância de cache criada e salve. Isso finalizará a configuração do cache.

![configure cache](images/25-configure-cache.png)

#### Configuração do healthcheck

26. Acesse por SSH a instância EC2 e execute o comando `sudo /root/moodle-scripts/after-moodle-installation.sh`.

![configure healthcheck](images/26-configure-healthcheck.png)

27. Após finalização do script, accesse a URL do load balancer usando a rota */admin/settings.php?section=tool_heartbeat*. Todos os itens devem estar marcados com ok. Apenas siga em frente.

![configure healthcheck](images/27-configure-healthcheck.png)

28. Clique em "verificar atualizações disponíveis". Se tiver alguma, siga as orientações na tela para realizar sua instalação.

![configure healthcheck](images/28-configure-healthcheck.png)

29. Após checar as atualizações, siga em frente.

![configure healthcheck](images/29-configure-healthcheck.png)

30. Confirme.

![configure healthcheck](images/30-configure-healthcheck.png)

31. Preencha o que achar necessário.

![configure healthcheck](images/31-configure-healthcheck.png)

32.  Para validar o processo, acesse a rota */admin/tool/heartbeat/*. Se for retornada a tela abaixo, a configuração do plugin Moodle Healthcheck foi finalizada com sucesso.

![configure healthcheck](images/32-configure-healthcheck.png)

#### Geração do AMI

33.  Na console da AWS, acesse a instância criada e, em "ações", crie sua imagem.

![generate ami](images/33-generate-ami.png)

34. Preencha os campos necessários e execute.

![generate ami](images/34-generate-ami.png)

35. Após a imagem estar com status disponível, copie seu AMI id.

![generate ami](images/35-generate-ami.png)

#### Atualização da stack

36. Volte no CloudFormation e delete a segunda stack (a que usa o arquivo [generate-moodle-ami-aws.yml](generate-moodle-ami-aws.yml)).

![update stack](images/36-update-stack.png)

37. Em seguida, volta na primeira stack (a que usa o arquivo [moodle-infra-aws.yml](moodle-infra-aws.yml)) para atualizá-la.

![update stack](images/37-update-stack.png)

38. Mantenha o template atual e prossiga.

![update stack](images/38-update-stack.png)

39. Dos parâmetros exibidos, altere LaunchTemplateVersion para 2, MoodleAMIImageID para o AMI id gerado e NumberOfInstances para o número preferível de instâncias a serem executadas. Finalize o processo de atualização da stack.

![update stack](images/39-update-stack.png)

#### Acesso ao Moodle

40. Acesse a URL do load balancer e prossiga mesmo com a notificação de conexão insegura.

![access moodle](images/40-access-moodle.png)

41. Se for exibida a tela inicial do Moodle, o processo completo foi finalizado com sucesso. Para validar, acesse "login".

![access moodle](images/41-access-moodle.png)

42. Preencha as credenciais da conta de administrador criada.

![access moodle](images/42-access-moodle.png)

43. A tela inicial da conta será exibida.

![access moodle](images/43-access-moodle.png)

#### Remoção de recursos

Para remover todos os recursos usados neste tutorial, além de remover as stacks, é necessário checar outros recursos que não são deletados junto com ela:
- RDS Snapshot: gerado na exclusão do RDS
- CloudWatch Log Groups: logs do Nginx
- AMI: imagens geradas
- EBS Snapshot: gerados para uso do AMI
- Certificado: criado no início do tutorial
- S3: arquivos de upload do CloudFormation

---

fim