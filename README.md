# SistemasOperacionais_TP3_SistemasdeArquivos
TP3 - Sistemas de Arquivos - Disciplina Sistemas Operacionais - DCC605 - UFMG
LINK do TP: http://homepages.dcc.ufmg.br/~cunha/teaching/20152/os/tpfs.html


<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
<title>DCC605/2015-1 --- TP3: DCC605FS</title>
<link rel="stylesheet" href="tps.css" type="text/css" />
</head>
<body>
<h1>DCC605/2015-1 --- TP3: Sistema de Arquivo DCC605FS</h1>

<p><b>Distribuído: quarta-feira, 10 de junho, 2015.<br/>
<!--Modificado: quarta-feira, 29 de maio, 2014.<br/>-->
Data de entrega: quarta-feira, 8 de julho, 2015.<br/></b></p>

<h2>Introdução</h2>

<p>Neste trabalho você irá implementar as primitivas de um sistema de
arquivos, o DCC605FS.  O DCC605FS é armazenado dentro de um arquivo no sistema
operacional.  Da mesma forma que você pode abrir um arquivo compactado (Zip) e
navegar pelo seu conteúdo, um sistema operacional poderá montar um arquivo
contendo um sistema de arquivos DCC605FS e navegar pelo seu conteúdo.</p>

<h2>Interface do sistema de arquivos</h2>

<p>Neste trabalho você irá implementar parte das funções do DCC605FS; em
particular, as funções para formatar um arquivo em disco com um novo DCC605FS
e para retornar um bloco livre no sistema de arquivos.  De forma mais
detalhada, as funções que você deve implementar são:

<div class="required">
<pre>
struct superblock * fs_format(const char *fname, uint64_t blocksize);
</pre>

<p>Constrói um novo sistema de arquivos no arquivo <tt>fname</tt>.  O arquivo
<tt>fname</tt> deve existir no sistema de arquivos do sistema operacional.  O
sistema de arquivos criado deve usar blocos de tamanho <tt>blocksize</tt>; o
número de blocos no sistema de arquivos deve ser calculado automaticamente
baseado no tamanho do arquivo <tt>fname</tt>.  O sistema de arquivos deve ser
inicializado com um diretório raiz vazio; o nome do diretório raiz deve ser
<tt>"/"</tt> como em sistemas UNIX.</p>

<p>Esta função retorna <tt>NULL</tt> em caso de erro e guarda em
<tt>errno</tt> o código de erro apropriado.  Se o tamanho do bloco for menor
que <tt>MIN_BLOCK_SIZE</tt>, a função falha e atribui <tt>EINVAL</tt> a
<tt>errno</tt>.  Se existir espaço em <tt>fname</tt> insuficiente para
armanzenar <tt>MIN_BLOCK_COUNT</tt> blocos, a função falha e atribui
<tt>ENOSPC</tt> a <tt>errno</tt>.</p>
</div>

<div class="required">
<pre>
struct superblock * fs_open(const char *fname);
</pre>

<p>Abre o sistema de arquivos em <tt>fname</tt> e retorna seu superbloco.
Retorna <tt>NULL</tt> em caso de erro e carrega o código de erro em
<tt>errno</tt>.  Caso o superbloco em <tt>fname</tt> não contenha o marcador
de sistemas de arquivo DCC605FS (<tt>0xdcc605f5</tt>), a função falha e
atribui <tt>EBADF</tt> a <tt>errno</tt>.

<p>Note que suas funções <tt>fs_format</tt> e <tt>fs_open</tt> não devem
permitir que o mesmo sistema de arquivo seja aberto duas vezes, para evitar
corrupção dos arquivos.  No caso do sistema de arquivos já estar aberto, estas
funções devem falhar e atribuir <tt>EBUSY</tt> a <tt>errno</tt>.
</div>

<div class="required">
<pre>
int fs_close(struct superblock *sb);
</pre>

<p>Fecha o sistema de arquivos apontado por <tt>sb</tt>.  Libera toda a
memória e recursos alocados à estrutura <tt>sb</tt>.  Retorna zero se não
houver erro.  Em caso de erro, um valor negativo é retornado e <tt>errno</tt>
indica o erro.  Se <tt>sb</tt> não tiver o marcador <tt>0xdcc605f5</tt>,
atribui <tt>EBADF</tt> a <tt>errno</tt>.
</div> 

<div class="required">
<pre>
uint64_t fs_get_block(struct superblock *sb);
</pre>

<p>Pega um ponteiro para um bloco livre no sistema de arquivos <tt>sb</tt>.  O
bloco retornado é retirado da lista de blocos livres do sistema de arquivos.
Retorna zero caso não existam mais blocos livres; retorna
<tt>(uint64_t)-1</tt> e atribui <tt>errno</tt> caso um erro ocorra.</p>
</div>

<div class="required">
<pre>
int fs_put_block(struct superblock *sb, uint64_t block);
</pre>

<p>Retorna <tt>block</tt> para a lista de blocos livres do sistema de arquivo
<tt>sb</tt>.  Retorna zero em caso de sucesso e um valor negativo em caso de
erro.  Código de erro, se ocorrer, é salvo em <tt>errno</tt>.
</div>

<div class="required">
<pre>
int fs_write_file(struct superblock *sb, const char *fname, char *buf, size_t cnt);
</pre>

<p>Escreve <tt>cnt</tt> bytes de <tt>buf</tt> no sistema de arquivos apontado
por <tt>sb</tt>.  Os dados serão escritos num arquivo chamado <tt>fname</tt>.
Retorna zero em caso de sucesso e um valor negativo em caso de erro; em caso de
erro, este será salvo em <tt>errno</tt> (p.ex., arquivo já existente, espaço em
disco insuficiente).</p> </div>

<div class="required">
<pre>
ssize_t fs_read_file(struct superblock *sb, const char *fname, char *buf,
                      size_t bufsz);
</pre>

<p>Lê os primeiros <tt>bufsz</tt> bytes do arquivo <tt>fname</tt> e coloca no
vetor apontado por <tt>buf</tt>.  Retorna a quantidade de bytes lidos em caso
de sucesso (pode ser menos que <tt>bufsz</tt> se o arquivo for menor que
<tt>bufsz</tt>) e um valor negativo em caso de erro.  Em caso de erro a
variável <tt>errno</tt> deve ser utilizada para indicar qual erro aconteceu.
</p> </div>

<div class="required">
<pre>
int fs_delete_file(struct superblock *sb, const char *fname);
</pre>

<p>Remove o arquivo chamado <tt>fname</tt> do sistema de arquivos apontado por
<tt>sb</tt> (os blocos associados ao arquivo devem ser liberados).  Retorna
zero em caso de sucesso e um valor negativo em caso de erro; em caso de erro,
este será salvo em <tt>errno</tt> (p.ex., arquivo não encontrado).</p> </div>

<h2>Estruturas do sistema de arquivos</h2>

Para facilitar a codificação do DCC605FS, várias estruturas de dados já foram
definidas.  Você deve manter estas estruturas de dados como definidas para
permitir interoperação de diferentes implementações do DCC605FS.  Para
facilitar, você pode assumir que o DCC605FS só será utilizado em computadores
<i>little-endian</i> como o Intel x86 e x64.

<div class="required">
<pre>
struct superblock {
	uint64_t magic; /* 0xdcc605f5 */
	uint64_t blks; /* number of blocks in the filesystem */
	uint64_t blksz; /* block size (bytes) */
	uint64_t freeblks; /* number of free blocks in the filesystem */
	uint64_t freelist; /* pointer to free block list */
	uint64_t root; /* pointer to root directory's inode */
	int fd; /* file descriptor for the filesystem image */
};
</pre>

<p>O superbloco é sempre armazenado no primeiro bloco do arquivo onde o
DCC605FS foi criado.  Os primeiros 64bits (campo <tt>magic</tt>) devem contar
o valor <tt>0xdcc605f5</tt>.  O campo <tt>root</tt> deve conter um ponteiro
para o bloco que armazena o diretório raiz do sistema de arquivo.  O inteiro
<tt>fd</tt> é utilizado para armazenar o descritor do arquivo que contém o
sistema de arquivo em disco.
</div>

<div class="required">
<pre>
struct inode {
	uint64_t mode;
	uint64_t parent;
	/* if =mode does not contain IMCHILD, then =parent points to the
	 * directory that contains this inode.  if =mode contains IMCHILD,
	 * then =parent points to the first inode (i.e., the inode without
	 * IMCHILD) for the entity represented by this inode. */
	uint64_t meta;
	/* if =mode does not contain IMCHILD, then meta points to this inode's
	 * metadata (struct iinfo).  if =mode contains IMCHILD, then meta
	 * points to the previous inode for this inode's entity. */
	uint64_t next;
	/* if this file's date block do not fit in this inode, =next points to
	 * the next inode for this entity; otherwise =next should be zero. */
	uint64_t links[];
	/* if =mode contains IMDIR, then entries in =links point to inode's
	 * for each entity in the directory.  otherwise, if =mode contains
	 * IMREG, then entries in =links point to this file's data blocks. */
};
</pre>

<p>A mesma estrutura <tt>inode</tt> acima é utilizada para armazenar
diretórios e arquivos.  Em diretórios, os <tt>links</tt> apontam para inodes
das entidades (arquivos e diretórios) contidos no diretório.  Em arquivos, os
<tt>links</tt> são <i>ordenados</i> e apontam para os blocos contendo os dados
do arquivo.  O campo <tt>mode</tt> diferencia entre os dois casos com as
máscaras:</p>

<pre>
#define IMREG 1   /* regular inode (normal file) */
#define IMDIR 2   /* directory inode */
</pre>

<p>Diretórios com muitas entidades e arquivos muito grandes podem requerer
quantidade de entradas no vetor <tt>links</tt> maior que disponível em um
único bloco.  Neste caso, o DCC605FS cria uma lista encadeada de inodes para
representar a entidade.  O campo <tt>next</tt> aponta para o próximo inode na
representação da entidade (ou tem o valor zero se o inode for o último).
Todos os inodes de uma entidade que não são o primeiro possuem a constante
<tt>IMCHILD</tt> em seu campo <tt>mode</tt>:</p>

<pre>
#define IMCHILD 4 /* child inode */
</pre>

<p>A semântica dos campos <tt>parent</tt> e <tt>meta</tt> de um inode varia
como dependendo se o inode for <tt>IMCHILD</tt> ou não.  Para inodes
<tt>IMCHILD</tt> o campo <tt>parent</tt> aponta para o primeiro inode da
entidade (o inode que não é <tt>IMCHILD</tt>); o campo <tt>meta</tt> aponta
pada o inode anterior na representação da entidade.  Para inodes que não são
<tt>IMCHILD</tt> o campo <tt>parent</tt> aponta para o inode do diretório que
contém a entidade; o campo <tt>meta</tt> aponta pada o inode que contém os
metadados do arquivo (<tt>struct nodeinfo</tt>).</p>

</div>

<div class="required">
<pre>
struct nodeinfo {
	uint64_t size;
	/* for files (mode IMREG), =size should contain the size of the file in 
	 * bytes.  for directories (mode IMDIR), =size should contain the
	 * number of files in the directory. */
	uint64_t reserved[7];
	/* reserving some space to implement security and ownership in the
	 * future. */
	char name[];
	/* remainder of block used to store this entity's name. */
};
</pre>

<p>A estrutura <tt>struct nodeinfo</tt> armazena metadados de uma entidade.
Por enquanto só armazenamos o tamanho e o nome da entidade.  Para arquivos
(<tt>IMREG</tt>), <tt>size</tt> deve conter o tamanho do arquivo em bytes.
Para diretórios (<tt>IMDIR</tt>), <tt>size</tt> deve conter o número de
entidade no diretório.  A cadeia de caracteres em <tt>name</tt> deve terminar
com o caractere nulo.</p>
</div>

<div class="required">
<pre>
struct freepage {
	uint64_t next;
	/* link to next freepage; or zero if this is the last freepage */
	uint64_t count;
	uint64_t links[];
	/* remainder of block used to store links to free blocks.  =count
	 * counts the number of elements in links, stored from links[0] to
	 * links[counts-1]. */
};
</pre>

<p>A <tt>struct freepage</tt> serve para armazenar os blocos de dados livres.
Como inodes, as páginas contendo os blocos de dados livres são encadeadas
através dos campos <tt>next</tt>.  O vetor <tt>links</tt> contém ponteiros
para blocos livres no sistema de arquivos.  O campo <tt>count</tt> contém o
número de blocos livres <i>nesta</i> página e serve para indexar o vetor
<tt>links</tt>.</p>
</div>

<p><b>Importante:</b> Os ponteiros para blocos no sistema de arquivo têm
64bits e contam o número de blocos no sistema de arquivo a partir de zero
(<i>não</i> contam número de bytes).</p>

<p><b>Importante:</b> A manutenção dos blocos livres não deve utilizar
quantidade arbitrariamente grande de blocos no disco.  Em particular, o <a
	href="dcc605fs/main.c">teste padronizado</a> confere se a quantidade de
blocos utilizados num sistema de arquivos vazio é menor que seis; idealmente, a
manutenção de blocos livres deve utilizar os blocos <em>livres</em>.</p>

<h2>Desenvolvimento e Avaliação</h2>

<p>Você pode baixar o arquivo <tt><a href="dcc605fs/fs.h">fs.h</a></tt> contendo
as definições acima.  Sua avlaiação será feita através de um <a
href="dcc605fs/main.c">teste padronizado</a> que você pode utilizar para
testar seu código.</p>

<p>Você deve entregar uma documentação de no máximo 4 páginas (2 folhas) em fonte
10 pontos, descrevendo (<i>com figuras</i>) a organização inicial do seu
sistema de arquivos e a organização das páginas de blocos livres (<i>com
figuras também</i>).  Sua documentação também deve discutir (e
justificar) as escolhas realizadas.</p>

<p>Se você identificar falhas ou ambiguidades ou problemas nesta
especificação, descrevê-las ao professor pode resultar na atribuição de pontos
extras.  Note que a estrutura inicial do sistema de arquivos e os algoritmos
para controle do espaço livre foram deixados em aberto intencionalmente.</p>

<h2>Extras</h2>

A implementação de gerência de diretórios será avaliada com nota extra.  Em
particular as seguintes funções:

<div class="required">
<pre>
int fs_mkdir(struct superblock *sb, const char *dname);
</pre>

<p>Cria um diretório no caminho <tt>dpath</tt>.  O caminho <tt>dpath</tt> deve
ser absoluto (começar com uma barra (<tt>/</tt>)).  Retorna zero em caso de
sucesso e um valor negativo em caso de erro; em caso de erro, este será salvo
em <tt>errno</tt> (p.ex., diretório já existente, espaço em disco
insuficiente).</p> </div>

<div class="required">
<pre>
char * fs_list_dir(struct superblock *sb, const char *dname);
</pre>

<p>Retorna um string separado com o nome de todos os elementos (arquivos e
diretórios) no diretório <tt>dname</tt>, os elementos devem estar separados por
espaço.  Os diretórios devem estar indicados com uma barra (<tt>/</tt>) ao
final do nome.  A ordem dos arquivos no string retornado não é relevante.  Por
exemplo, se um diretório contém três elementos--um diretório <tt>d1</tt> e dois
arquivos <tt>f1</tt> e <tt>f2</tt>--esta função deve retornar:</p>

<pre>
d1/ f1 f2
<pre> </div>



</body>
</html>

