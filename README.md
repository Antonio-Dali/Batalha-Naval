# Batalha-Naval
Um primeiro programa em C


#include<stdio.h>
#include<stdlib.h>

#define MAXLINHAS     20
#define MAXCOLUNAS    20
#define MAXNOME       64
#define TRUE           1
#define FALSE          0
#define SUBMARINO     'S'
#define DESTROYER     'D'
#define CRUZADOR      'C'
#define PORTA_AVIAO   'P'
#define HIDRO_AVIAO   'H'
#define BARCO         'B'
#define EXPLOSAO      '*'
#define AGUA          'a'
#define RASTRO        '.'
#define ESQUERDA      'e'
#define BAIXO         'b'
#define DIREITA       'd'
#define CIMA          'c'

/*                                         */
/*              Seção leia mapa            */
/*                                         */

/*  inicializa_gerador: recebe um inteiro semente e inicializa a
    semente do gerador de numeros aleatorios com semente.*/
void inicializa_gerador(int semente)
{
  srand(semente);
}

/* leia_ate_barra_n: le caracteres do arquivo associado a
  arq ate' que um fim-de-linha '\n' seja lido. */
void leia_ate_barra_n (FILE *arq)
{
  char ch; /* usado na leitura dos caracteres */

  /* le os caracteres ate' encontrar o fim-de-linha */
  fscanf(arq, "%c", &ch);
  while (ch != '\n')
    {
      fscanf(arq, "%c", &ch);
    }
}

/* lê dentro da linha 0, coluna por coluna ate encontrar o elemento B*/
int coluna_inicial_barco(char mapa[MAXLINHAS][MAXCOLUNAS], int no_col)
{
  int col, c;
  char B;
  c=0;

  for(col=0; col<=no_col; col++)
  {
    if (mapa[0][col]=='B')
    {
      c=col+1;
    }
  }

  return c;
}

/* leia_mapa: le de um arquivo (cujo nome eh fornecido pelo usuario
   via scanf) tres inteiros lin, col, codigo e
   uma matriz de caracteres com lin linhas e col colunas.
   A funcao inicializa a semente do gerador de numeros aleatorios
  com o valor de codigo e devolve em *no_linhas o valor de lin,
   em *no_colunas o valor de col e em mapa a matriz de caracteres
   lida. Alem disso a funcao devolve em *c_barco uma valor tal que
   mapa[1][*cbarco] == 'B' (posicao inicial do barco).

   Observacao: nesta funcao esta o unico lugar no programa
   em que a semente do gerador de numeros aleatorios e'
   inicializada. (!) */
void leia_mapa (int *no_linhas, int *no_colunas, char mapa[MAXLINHAS][MAXCOLUNAS], int *c_barco)
{
  FILE *arq;   /* arquivo que contem o mapa */
  char nome_arquivo[MAXNOME]; /* nome do arquivo que contem o mapa */
  int no_l;    /* numero de linha do mapa */
  int no_c;    /* numero de colunas do mapa */
  int c_bar;   /* coluna inicial do barco */
  int i;       /* indice de linhas */
  int j;       /* indice de colunas */
  int codigo;  /* semente do gerador de numeros aleatorios */

  printf("MSG do QG: preparando para leitura do mapa.\n");

  /* 1. leia o nome do arquivo */
  printf("MSG do QG: forneca nome do arquivo que contem o mapa: ");
  scanf("%s", nome_arquivo);

  /* 2. abra o arquivo para leitura */
  arq = fopen(nome_arquivo,"r");
  if (arq == NULL)
    {
      printf("MSG do QG: erro na abertura do arquivo %s.\n", nome_arquivo);
      printf("MSG do QG: MISSAO ABORTADA!\n\n");
      exit(-1);
    }
  printf("MSG do QG: arquivo %s foi encontrado.\n", nome_arquivo);

  /* 3. leia o numero de linha n e o numero de coluna m do mapa */
  printf("MSG do QG: mapa esta sendo lido do arquivo %s.\n", nome_arquivo);
  fscanf(arq, "%d %d %d", &no_l, &no_c, &codigo);
  leia_ate_barra_n(arq);
  printf("MSG do QG: mapa tem %d linhas e %d colunas.\n", no_l, no_c);
  printf("MSG do QG: codigo de ataque: %d\n", codigo);


   /* 4. inicializa o gerador de numeros aleatorios */
  inicializa_gerador(codigo);

  /* 5. leia o mapa */
  for (i = 0; i <= no_l; i++)
    {
      for (j = 0; j < no_c; j++)
        {
          fscanf(arq, "%c", &mapa[i][j]);
        }
      leia_ate_barra_n(arq); /* le os caracteres ate' o fim-de-linha*/
    }

  /* 6. determine a posicao inicial do barco */
  c_bar = coluna_inicial_barco(mapa, no_c);

  printf("MSG do QG: posicao inicial do barco e' (1,%d).\n", c_bar);

  /* 7. feche o arquivo */
  fclose(arq);
  printf("MSG do QG: mapa lido com sucesso.\n");

 /* 8. devolva o numero de linhas, colunas e coluna do barco */
  *no_linhas  = no_l;
  *no_colunas = no_c;
  *c_barco    = c_bar;
}

/*                                                    */
/*              Seção escreva mapa secreto            */
/*                                                    */

/*Esta função cria a cercadura composta por +---+ que é exibido no mapa*/
void vetorcercadura ( int no_colunas, FILE *arq)
{
  int col;

  fprintf(arq, "  +");

  for (col = 0; col < no_colunas; col++)
  {
    fprintf(arq, "---+");
  }
  fprintf(arq, "\n");
}

/*Essa função cria um vetor que escreverá a coluna com numeração e a
disponibilização dos caracteres que representão as embarcações*/
void vetorcerc_central ( int no_colunas, char mapa[MAXLINHAS][MAXCOLUNAS], int l, int lin, FILE *arq)
{
  int  col;

  if (l<10)
  {
    fprintf(arq, "%d |", l);
  }
  else
  {
    fprintf(arq, "%d|", l);
  }

  for ( col = 0; col < no_colunas; col++)
  {
    fprintf(arq, " %c |", mapa[lin][col]);
  }
  fprintf(arq, "\n");
}

/* escreva_mapa_arquivo: recebe um mapa de no_linhas linhas e
   no_colunas colunas e escreve o mapa no arquivo secreto.dat */
void escreva_mapa_arquivo (int no_linhas, int no_colunas, char mapa[MAXLINHAS][MAXCOLUNAS])
{
  FILE *arq;
  int i, lin, l; /* indice de linhas */
  int j, col, cont, c; /* indice de colunas */

  /* abra arquivo para escrita */
  arq = fopen("secreto.dat","w");
  if (arq == NULL)
    {
      printf("MSG do QG: erro na criacao do arquivo secreto.dat.\n");
      printf("MSG do QG: MISSAO ABORTADA!\n\n");
      exit(-1);
    }

  /* escreva o cabecalho no arquivo */
  fprintf(arq, "    MAPA SECRETO DA BATALHA\n\n");
  fprintf(arq, "    no. linhas = %d     no. colunas = %d\n\n", no_linhas,  no_colunas);

/* imprime o mapa no arquivo secreto.dat */
/*Esse trecho printa a numeração das colunas*/
  fprintf(arq, "    ");/* esse print imprime os 4 espaços antes do primeiro numero*/

  for (j = 1; j <= no_colunas; j++)
  {
    if (j<9)/*Para numeros de 1 digito, imrime 1 numero e 1 espaço*/
    {
      fprintf(arq, "%d   ", j);
    }
    else  /*Para numeros de 2 digitos, imprime o numero de dois digitos sem espaço*/
    {
      fprintf(arq, "%d  ", j);
    }
  }
  fprintf(arq, "\n");

/*Esse vetor cria uma linha que será printada na imagem do mapa composta por +---+*/

  vetorcercadura ( no_colunas, arq);

  l=1;
  c=0;
  for ( cont = 0; cont < no_linhas; cont++)
  {
/*Vetor "vetorcerc_central" é o vetor que escreve a linha do mapa onde estão representadas as embarcaçoes,
esse vetor fica entre dois vetores +---+*/
    vetorcerc_central ( no_colunas, mapa, l, c, arq);
    vetorcercadura ( no_colunas, arq);
    l++;
    c++;
  }

  /* feche o arquivo */
  fprintf(arq, "\n\nFIM DO ARQUIVO.");
  fclose(arq);
}

/*                                                    */
/*              Seção escreva mapa na Tela            */
/*                                                    */

/*Esta função cria a cercadura composta por +---+ que é exibido no mapa*/
void cercaduratela ( int no_colunas)
{
  int col;

  printf("  +");

  for (col = 0; col < no_colunas; col++)
  {
    printf("---+");
  }
  printf("\n");
}

/*Essa função cria um vetor que escreverá a coluna com numeração e a
disponibilização dos caracteres que representão as embarcações trocados por espaço*/
void cerc_centraltela ( int no_colunas, char mapa[MAXLINHAS][MAXCOLUNAS], int l, int cont)
{
  int  c;

  if (l<10)
  {
    printf("%d |", l);/**/
  }
  else
  {
    printf("%d|", l);/**/
  }

  for ( c = 0; c < no_colunas; c++)/*Comando que realiza o preenchiento das colunas*/
  {
    if (mapa[cont][c]=='S' || mapa[cont ][c]=='D' || mapa[cont][c]=='C' || mapa[cont][c]=='P' || mapa[cont][c]=='H')/**Se o mapa apresenta
 um desses caracteres, é impresso espaço em branco*/
    {
      printf("   |");
    }
    else/*Se não é nenhum deles o mapa é impresso normalmente seguindo a matriz mapa*/
    {
      printf(" %c |", mapa[cont][c]);
    }

  }
  printf("\n");
}


/* escreva_mapa_tela: recebe um matriz mapa de no_linhas linhas e
   no_colunas colunas e escreve mapa na tela exibindo o caractere
   ' ' (espaço) no lugar dos caracteres 'S', 'D', 'C', 'P' e 'H'. */
void escreva_mapa_tela (int no_linhas, int no_colunas, char mapa[MAXLINHAS][MAXCOLUNAS])
{
  int i, l, cont;
  int j;

  printf("MSG do QG: segue mapa da batalha.\n\n");
  printf("    MAPA DA BATALHA\n\n" );
  /*Trecho para descrever o numero de colunas*/

  /*Esse trecho printa a numeração das colunas*/
  printf("    ");/* esse print imprime os 4 espaços antes do primeiro numero*/

  for (j = 1; j <= no_colunas; j++)
  {
    if (j<9)/*Para numeros de 1 digito, imrime 1 numero e 1 espaço*/
    {
      printf("%d   ", j);
    }
    else  /*Para numeros de 2 digitos, imprime o numero de dois digitos sem espaço*/
    {
      printf("%d  ", j);
    }
  }
  printf("\n");

  cercaduratela( no_colunas);
  l=1;
  for ( cont = 0; cont < no_linhas; cont++)
  {
  /*Vetor "vetorcerc_central" é o vetor que escreve a linha do mapa onde estão representadas as embarcaçoes,
  esse vetor fica entre dois vetores +---+*/
    cerc_centraltela ( no_colunas, mapa, l, cont);
    cercaduratela ( no_colunas);
    l++;
  }
  printf("\n\n");
}

/*                                                     */
/*              Seção movimentação do Barco            */
/*                                                     */

int movimento_barco ( int no_linhas, int no_colunas, char Cmapa[MAXLINHAS][MAXCOLUNAS], int movimento)
{
 int l, lin, linha;
 int c, col, coluna;
 char ch;

 printf("MSG do QG: escolha movimento ('e'=esquerda,'b'=baixo,'d'=direita,'c'=cima):");
 scanf(" %c", &ch);
 if (ch=='f')
 {
   exit(-1);
 }

/*Localiza a posição exata do barco*/
 for ( l = 0; l <= no_linhas; l++)
 {
   for ( c = 0; c <= no_colunas; c++)
   {
     if (Cmapa[l][c] == 'B')
     {
       linha=l;
       lin=l;/*guarda a posição inicial em linha caso necessário(o barco não se mexa)
       e permite a manipulação de lin*/
       coluna=c;
       col=c;/*guarda a posição inicial em coluna caso necessário(o barco não se mexa)
       e permite a manipulação de col*/
     }
   }
 }

/*Altera a posição do barco de acordo com a movimentação do jogador*/
/*Os proximos 4 ifs alteram a posição de linha e coluna do barco para
cada comando (e,d,c,b,) a menos que o barco saia do mapa*/
 if (ch=='e')/*Se o jogada foi para esquerda*/
 {
   if (col!=0)
   {
     col--;
   }
 }
 if (ch=='d')/*Se o jogada foi para direita*/
 {
   if (col <= (no_colunas-2))
   {
     col++;
   }
 }
 if (ch=='c')/*Se o jogada foi para cima*/
 {
   if (lin!=0)
   {
     lin--;
   }
 }
 if (ch=='b')/*Se o jogada foi para baixo*/
 {
   if (lin<no_linhas)
   {
     lin++;
   }
 }
 /*Se o barco encontra uma embarcação então as variaveis retornarm ao valor do barco para que ele não altere sua posição*/
 if (Cmapa[lin][col]=='S' || Cmapa[lin][col]=='D' || Cmapa[lin][col]=='C' || Cmapa[lin][col]=='P' || Cmapa[lin][col]=='H')
 {
   lin++;/*Valores com os indices corretos para impressão*/
   col++;/*Valores com os indices corretos para impressão*/
   printf("MSG do QG: ha uma embarcacao na posicao (%d,%d).\n", lin, col);
   lin=linha;/*Corrige lin e col para os valores onde o barco estava para que ele não se movimente usando o trecho a seguir*/
   col=coluna;
 }

 c=col+1;/*Valores com os indices corretos para impressão*/
 l=lin+1;/*Valores com os indices corretos para impressão*/

 /*Atualiza a posição do barco*/
 if (col!=coluna || lin!=linha)
 {
   Cmapa[linha][coluna]='.';
   Cmapa[lin][col]='B';
   printf("MSG do QG: moveu-se para (%d,%d).\n", l, c );
   movimento=0;/*Se o barco se movimentar, zera o contador para que ele so conte sucessivas vezes sem se mover*/
 }
 else
 {
   printf("MSG do QG: ficou parado em (%d,%d).\n", l, c );
   movimento++;/*conta quantas vezes o barco ficou parado*/
 }
 return movimento;
}

/*                                                       */
/*              Seção função tiros aleatorios            */
/*                                                       */

/* no_aletorio: recebe um inteiro n e devolve um inteiro no intervalo [1..n]*/
int no_aleatorio (int n)
{
  return  1 + (rand()/(RAND_MAX+1.0))*n;
}

/*sorteie_posicao: recebe um inteiro no_linhas e um inteiro
  no_colunas e devolve em *linha um numero aleatorio no intervalo
 [1..no_linhas] e devolve em *coluna um numero aleatorio no
 intervalo [1..no_colunas].*/

void sorteie_posicao (int no_linhas, int no_colunas, int *linha, int *coluna)
{
  *linha  = no_aleatorio(no_linhas);
  *coluna = no_aleatorio(no_colunas);
}

/* acertou_embarcacao: recebe um caractere e devolve TRUE se o
  caractere corresponde a uma embarcacao nao afundada;
  caso contrario a funcao devolve FALSE.*/
int acertou_embarcacao(char ch)
{
  if (ch=='S' || ch=='D' || ch=='C' || ch=='P' || ch=='H')
  {
    ch=TRUE;
  }
  else
  {
    ch=FALSE;
  }
  return ch;
}

/*
 * afunda_embarcacao: recebe um posicao (linha, coluna)
 *   que contem uma embarcacao e destroi a embarcacao,
 *   colocando o caractere '*' na posicao atingida e trocando os
 *   demais caracteres de maiusculo para minusculo.
 */
void afunda_embarcacao(int linha,int coluna, char mapa[MAXLINHAS][MAXCOLUNAS])
{
  int l, c;
  int lin, col;
  int cont;



  /*Como os contadores verificaram casas a direita da posição atingida da embarcação, o comando while repete
  mais uma vez a sequencia no ponto atingido a fim de verificar se a embarcação tem mais letras a esquerda*/
  for ( cont = 0; cont < 2; cont++)
  {
  /*Define a casa em que esta a embarcação atingida como casa central*/
    lin=(linha-1);
    col=(coluna-1);
    /*analisa as oito casas em volta da casa central e caso seja parte da embarcação passa para letra minuscula*/
    for ( l=lin-1; l <= lin+1; l++)
    {
      for ( c=col-1; c <= col+1; c++)
      {
        if (mapa[l][c]=='D' || mapa[l][c]=='C' || mapa[l][c]=='P' || mapa[l][c]=='H')
        {
          mapa[l][c]=((mapa[l][c])+32);/*transforma letra maiuscula para minuscula*/
          /*redefine parametros para que a letra que foi transformada em minuscula seja a casa centro */
          lin=l;
          l=lin-2;
          col=c;
          c=col+2;
        }
      }
    }
  }
}

 void tiroteio ( int no_linhas, int no_colunas, char mapa[MAXLINHAS][MAXCOLUNAS])
 {
   int linha, coluna;
   int l, lin;
   int c, col;
   int cont, emb;
   char ch;

   /*Localiza o barco*/
   for ( l = 0; l <= no_linhas; l++)
   {
     for ( c = 0; c <= no_colunas; c++)
     {
       if (mapa[l][c] == 'B')
       {
         lin=l+1;/*Valores com os indices corretos para impressão*/
         col=c+1;/*Valores com os indices corretos para impressão*/
       }
     }
   }

   for (cont = 0; cont < 3; cont++)
   {
     sorteie_posicao(no_linhas, no_colunas, &linha, &coluna);

     if (mapa[linha-1][coluna-1]==' ' || mapa[linha-1][coluna-1]=='a' || mapa[linha-1][coluna-1]=='.')
     {
       printf("MSG do QG: posicao (%d,%d), CHUA ... AGUA\n", linha, coluna);
       mapa[linha-1][coluna-1]='a';
     }

     if (mapa[linha-1][coluna-1]=='B')
     {
       printf("MSG do QG: posicao (%d,%d), BUUMMM\n", lin, col);
       printf("MSG do QG: posicao (%d,%d), opsssss... voce foi atingido!\n", lin, col);
       printf("MSG do QG: embarcacao afundada.\n");
       mapa[linha-1][coluna-1]='X';
       cont=3;
     }

     ch = mapa[linha-1][coluna-1];
     emb = acertou_embarcacao(ch);

     if (mapa[linha-1][coluna-1]=='*' || mapa[linha-1][coluna-1]=='d' || mapa[linha-1][coluna-1]=='c' || mapa[linha-1][coluna-1]=='p' || mapa[linha-1][coluna-1]=='h')
     {
       printf("MSG do QG: posicao (%d,%d), embarcação atingida novamente!\n", linha, coluna);
     }

     if (emb==TRUE)
     {
       printf("MSG do QG: posicao (%d,%d), BUUMMM\n", linha, coluna);

       if (mapa[linha-1][coluna-1]=='S')
       {
         printf("MSG do QG: posicao (%d,%d), submarino atingido!\n", linha, coluna);
       }
       if (mapa[linha-1][coluna-1]=='D')
       {
         printf("MSG do QG: posicao (%d,%d), destroyer atingido!\n", linha, coluna);
         afunda_embarcacao (linha, coluna, mapa);
       }
       if (mapa[linha-1][coluna-1]=='C')
       {
         printf("MSG do QG: posicao (%d,%d), cruzador atingido!\n", linha, coluna);
         afunda_embarcacao (linha, coluna, mapa);
       }
       if (mapa[linha-1][coluna-1]=='P')
       {
         printf("MSG do QG: posicao (%d,%d), porta-avioẽs atingido!\n", linha, coluna);
         afunda_embarcacao (linha, coluna, mapa);
       }
       if (mapa[linha-1][coluna-1]=='H')
       {
         printf("MSG do QG: posicao (%d,%d), hidro-aviao atingido!\n", linha, coluna);
         afunda_embarcacao (linha, coluna, mapa);
       }
       mapa[linha-1][coluna-1]='*';
     }
   }
  }

/*                                                           */
/*              Seção função mapa secreto na tela            */
/*                                                           */

/*Essa função cria um vetor que escreverá a coluna com numeração e a
disponibilização dos caracteres que representão as embarcações trocados por espaço*/
void cerc_centraltela_secreto ( int no_colunas, char mapa[MAXLINHAS][MAXCOLUNAS], int l, int cont)
{
  int  c;

  if (l<10)
  {
    printf("%d |", l);/**/
  }
  else
  {
    printf("%d|", l);/**/
  }

  for ( c = 0; c < no_colunas; c++)/*Comando que realiza o preenchiento das colunas*/
  {
    printf(" %c |", mapa[cont][c]);
  }
  printf("\n");
}


void escreva_mapa_secreto_tela (int no_linhas, int no_colunas, char mapa[MAXLINHAS][MAXCOLUNAS])
{
  int i, l, cont;
  int j;

  printf("    MAPA SECRETO DA BATALHA\n\n");
  printf("    no. linhas = %d     no. colunas = %d\n\n", no_linhas, no_colunas );
  /*Trecho para descrever o numero de colunas*/

  /*Esse trecho printa a numeração das colunas*/
  printf("    ");/* esse print imprime os 4 espaços antes do primeiro numero*/

  for (j = 1; j <= no_colunas; j++)
  {
    if (j<9)/*Para numeros de 1 digito, imrime 1 numero e 1 espaço*/
    {
      printf("%d   ", j);
    }
    else  /*Para numeros de 2 digitos, imprime o numero de dois digitos sem espaço*/
    {
      printf("%d  ", j);
    }
  }
  printf("\n");

  cercaduratela( no_colunas);
  l=1;
  for ( cont = 0; cont < no_linhas; cont++)
  {
  /*Vetor "vetorcerc_central" é o vetor que escreve a linha do mapa onde estão representadas as embarcaçoes,
  esse vetor fica entre dois vetores +---+*/
    cerc_centraltela_secreto ( no_colunas, mapa, l, cont);
    cercaduratela ( no_colunas);
    l++;
  }
  printf("\n\n");
}

/*                                                     */
/*              Seção função principal main            */
/*                                                     */

int main()
{
  int no_linhas, lin, l, linhai;
  int no_colunas, col, c, colunai;
  int c_barco;
  char mapa[MAXLINHAS][MAXCOLUNAS];
  int fdjogo, cont, movimento;

  printf("MSG do QG: prepare-se para a missao.\n");

  leia_mapa( &no_linhas, &no_colunas, mapa, &c_barco);

  escreva_mapa_arquivo ( no_linhas, no_colunas, mapa);

  escreva_mapa_tela ( no_linhas, no_colunas, mapa);

  /*Complementa uma linha e coluna da matriz mapa com "a" após a ultima linha e coluna preenchidos*/
  for ( l= 0; l<=no_linhas ; l++)
  {
    mapa[l][no_colunas]='a';
  }
  for (c = 0; c < no_colunas; c++)
  {
    mapa[no_linhas][c]='a';
  }

  movimento=0;
  cont=no_linhas-1;
  fdjogo=0;

    printf("MSG do QG: preparando artilharia para lhe dar cobertura.\n");
    printf("MSG do QG: cuidado com fogo amigo...\n");
    printf("MSG do QG: preparados para iniciar bombardeio da frota inimiga.\n");
    printf("MSG do QG: permissao para iniciar a missao concedida.\n");
    /*Localiza o barco*/
    for ( l = 0; l <= no_linhas; l++)
    {
      for ( c = 0; c <= no_colunas; c++)
      {
        if (mapa[l][c] == 'B')
        {
          lin=l+1;/*Valores com os indices corretos para impressão*/
          col=c+1;/*Valores com os indices corretos para impressão*/
        }
      }
    }
    printf("MSG do QG: a sua posicao e' (%d,%d).\n", lin, col);
    printf("MSG do QG: Boa sorte!\n\n");

  while (fdjogo==0)
  {
    movimento = movimento_barco ( no_linhas, no_colunas, mapa, movimento);

    if (movimento>=3) /*Se o barco ficou parado tres vezes finaliza o jogo*/
    {
      fdjogo=1;
    }

    tiroteio ( no_linhas, no_colunas, mapa);

    escreva_mapa_tela ( no_linhas, no_colunas, mapa);

    escreva_mapa_arquivo ( no_linhas, no_colunas, mapa);

    for( l= 0; l < no_linhas; l++)
    {
      for ( c=0; c < no_colunas; c++)
      {
        if (mapa[cont][c]=='B' || mapa[l][c]=='X')
        {
          fdjogo=1;
          linhai=l;
          colunai=c;
        }
      }
    }
  }

  if (movimento>=3)
  {
    printf("MSG do QG: Você ficou parado em (%d,%d) 3 vezes consecutivas.\n", lin, col);
    printf("MSG do QG: VOCE FRACASSOU EM SUA MISSAO... FICA PARA A PROXIMA.\n\n\n");
    escreva_mapa_secreto_tela ( no_linhas, no_colunas, mapa);
    return 0;
  }

  if (mapa[linhai][colunai]=='X')
  {
    printf("MSG do QG: infelizmente voce foi destruido em (%d,%d).\n", linhai, colunai);
    printf("MSG do QG: VOCE FRACASSOU EM SUA MISSAO... FICA PARA A PROXIMA.\n\n\n");
    escreva_mapa_secreto_tela ( no_linhas, no_colunas, mapa);
    return 0;
  }

  printf("MSG do QG: Parabéns você concluiu sua missão!!!\n\n\n" );
  escreva_mapa_secreto_tela ( no_linhas, no_colunas, mapa);

  return 0;
}
