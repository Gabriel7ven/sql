# sql

 Vbs#572#45#Ujk

Up9dn3kpv7

\\10.30.58.44\inetpub\wwwroot\sicadhm\v1\Download\ceee\report\

select * from view_avaliar_poste
select * from servicO
SELECT SERVICO FROM PONTO_GEOGRAFICO WHERE PGF_BARRAMENTO='28253'

--POSTE--
select
p.pgf_utmx,
p.pgf_utmy,
p.servico,
p.pgf_barramento,
tp.tb_pg_descricao,
tc.descricao_m material,
tc.tb_al_id altura,
tc.tb_ef_id esforco
from
ponto_geografico                                p
inner join tipo_de_pg                           tp                     on tp.tb_pg_id=p.tb_pg_id
inner join tipo_de_caracteristica               tc                     on tc.tb_ca_id=p.tb_ca_id
where
p.servico ='1008202214400734785'

--INSTALACAO DE CARGA--
select
p.servico,
P.PGF_UTMX,
P.PGF_UTMY,
p.pgf_barramento,
eq.EQ_NR_PATRIMONIAL,
eq.EQ_KVAN,
eq.EQ_FASES,
eq.EQ_NR_PATRIMONIAL EQ_NR_PATRIMONIAL2,
eq.EQ_KVAN EQ_KVAN2,
eq.EQ_FASES EQ_FASES2


from ponto_geografico        p
inner join no                n on n.mslink_pg=p.mslink
inner join instalacao        i on i.no_id=n.no_id
inner join equipamento_sap   eq on eq.mslink_ins=i.mslink
where p.servico='1008202214263834761'



--INSTALACAO DE MANOBRA--
select
p.servico,
p.pgf_utmx,
p.pgf_utmy,
p.pgf_barramento,
I.TB_CDF_ID,
TI.TB_COD_REAL,
IM.INT_IND_POS_NORM,
I.TB_CDF_ID TB_CDF_ID2,
TI.TB_COD_REAL TB_COD_REAL2,
IM.INT_IND_POS_NORM INT_IND_POS_NORM2


from ponto_geografico              p
inner join no                      n  on n.mslink_pg=p.mslink
inner join instalacao              i  on i.no_id=n.no_id
inner join instalacao_manobra      im on im.mslink=i.mslink
INNER JOIN TIPO_DE_INSTALACAO      TI ON TI.TB_IN_ID=I.TB_IN_ID
where P.SERVICO ='1008202214263834761'

-----REDE MT---------
select 
TR.MSLINK
,p.PGF_BARRAMENTO barramento_de
,p2.PGF_BARRAMENTO barramento_pa
--,se.se_codigo
--,al.al_codigo
,TR.TB_CDF_ID
,TR.TB_TR_ID
,TC.DESCRICAO
,TR.TB_CDF_ID TB_CDF_ID2
,TR.TB_TR_ID TB_TR_ID2
,TC.DESCRICAO DESCRICAO2

,        'LINESTRING(' ||
         ROUND(p.PGF_UTMX) ||
         ' '||
         ROUND(p.PGF_UTMY) ||
         ','||
         ROUND(p2.PGF_UTMX)||
         ' '||
         ROUND(p2.PGF_UTMY)||
         ')' shape

from
trecho_de_rede tr
INNER JOIN TIPO_DE_CONDUTOR TC ON tr.TB_CND_ID=TC.TB_CND_ID
inner join no n on tr.NO_ID_DE=n.NO_ID
left join no n2 on tr.NO_ID_PA=n2.NO_ID
left join ponto_geografico p on n.MSLINK_PG=p.MSLINK 
left join PONTO_GEOGRAFICO p2 on n2.MSLINK_PG=p2.MSLINK
--inner join trecho_primario tp on tr.mslink=tp.mslink
--inner join alimentador al on tp.al_id=al.al_id
--inner join barramento_se ba on ba.ba_id=al.ba_id
--inner join subestacao se on se.se_id=ba.se_id

where se.se_codigo='&subestacao' and al.al_codigo='&alimentador'






-----CARIMBAR ALIMENTADOR--------
Insert into ALIMENTADOR_AMAPA (ALIMENTADOR,SERVICO) values ('XXXX_000','00000000000000000');
Insert into ALIMENTADOR_AMAPA (ALIMENTADOR,SERVICO) values ('XXXX_000','00000000000000000');
commit;






----CARIMBAR POSTE--------
update ponto_geografico set lg_id=1 where servico in(select servico from alimentador_amapa);
commit;





-------CORRENTE NOMINAL =0 ------
SELECT  IM.MSLINK,IM.TB_CAP_ID,TI.TB_IN_DESCRICAO
 FROM INSTALACAO_MANOBRA IM
 INNER JOIN TIPO_DE_INSTALACAO TI ON TI.TB_IN_ID=IM.TB_IN_ID 
 WHERE 
   (TI.TB_IN_DESCRICAO NOT IN ('Chave Fusível') AND IM.TB_CAP_ID =0)




------PRIMARIO DESENERGIZADO-----------------
select tr.mslink from trecho_de_rede tr,trecho_primario tp,no n, ponto_geografico pg

where tr.mslink=tp.mslink
      and (tr.no_id_de=n.no_id or tr.no_id_pa=n.no_id)
      and pg.mslink=n.mslink_pg
      and tp.trc_energizado ='N'
      and pg.lg_id=1    --------primario



-----------SECUNDARIO DESENERGIZADO----------
select tr.mslink from trecho_de_rede tr,trecho_secundario ts,no n, ponto_geografico pg

where tr.mslink=ts.mslink
      and (tr.no_id_de=n.no_id or tr.no_id_pa=n.no_id)
      and pg.mslink=n.mslink_pg
      and ts.mslink_ins is null
      and pg.lg_id=1 

---------MEDIDOR DESENERGIZADO-----------
SELECT ap.alimentador,C.CR_NUMERO,c.cr_medidor,p.mslink MSLINK_PG,P.PGF_BARRAMENTO
FROM CONSUMIDOR C 
INNER JOIN NO N ON C.NO_ID=N.NO_ID
INNER JOIN PONTO_GEOGRAFICO P ON P.MSLINK=N.MSLINK_PG
INNER JOIN ALIMENTADOR_AMAPA AP ON AP.SERVICO=P.SERVICO
INNER JOIN TRECHO_SECUNDARIO TS ON c.mslink_tr=TS.MSLINK
WHERE P.LG_ID=1 AND c.mslink_ins IS NULL AND ts.mslink_ins IS NULL




------------SLQ PARA MUDAR LOCALIZACAO DE PPG PARA  RURAL-----
UPDATE PONTO_GEOGRAFICO SET PGF_IND_LOC = 'R' WHERE MSLINK IN (
select mslink from view_avaliar_poste where avaliar like '%LOCALIZACAO%')




------SQL MUDAR PARA URBANO TEM QUE LISTAR OS MSLINKS---------
UPDATE PONTO_GEOGRAFICO SET PGF_IND_LOC = 'U' WHERE MSLINK IN (,,,,,,,,)





---------




--TIRAR PARA RAIO DE POSTE SÓ COM BT---=

update ponto_geografico set tb_pr_id= 0 where mslink in(

select mslink from ponto_geografico p where mslink not in
(
select
p.mslink
--p.tb_pr_id,
--tr.tb_tr_id
from

no n
left join ponto_geografico p on p.mslink=n.mslink_pg
left join trecho_de_rede tr on tr.no_id_de=n.no_id or tr.no_id_pa=n.no_id
where tr.tb_tr_id='PA'
) and tb_pr_id = 1
   
 )



-- DELETAR IP --
select mslink from ponto_geografico where pgf_barramento='5587'


delete ip where mslink in (
select mslink from ip where mslink_pg='129262808')



----- DELETAR INSTALAÇÃO ------------
delete instalacao 
where mslink in 
(
  SELECT 
       I.MSLINK
  FROM INSTALACAO I 
       INNER JOIN NO                  ON NO.NO_ID=I.NO_ID
       INNER JOIN PONTO_gEOGRAFICO PG ON PG.MSLINK=NO.MSLINK_PG
  WHERE 
       PG.PGF_BARRAMENTO = '143296'
)


----------- EQUIPAMENTO-----------


DELETE FROM EQUIPAMENTO_SAP WHERE MSLINK_INS IN 
(
  SELECT 
       I.MSLINK
  FROM INSTALACAO I 
       INNER JOIN NO                  ON NO.NO_ID=I.NO_ID
       INNER JOIN PONTO_gEOGRAFICO PG ON PG.MSLINK=NO.MSLINK_PG
  WHERE 
       PG.PGF_BARRAMENTO = '143296'


-------------------LISTAR FOTOS DE POR SERVICO------------------------------

select 
   -- d.arg_sequencia
    d.arg_nome_arquivo
    ,e.servico
   ,'\\10.7.1.45\OraFotoDir\CELPA\' || e.servico  PARCIAL
    ,'\\10.7.1.45\OraFotoDir\CELPA\' || e.servico || '\' || d.arg_nome_arquivo CONPLETO
    ,p.pgf_barramento
    from 
    digital_arquivo d
    
    inner join elemento_rede_digital e on e.arg_id=d.arg_id
    inner join ponto_coletado_grafico pc on pc.ponto_coletado_id=e.ponto_coletado_id
    inner join ponto_geografico p on pc.mslink=p.mslink
    
    where p.servico in ('04102022135002141924',
'04102022135034141925',
)
    order by p.pgf_barramento,d.arg_sequencia

--------------------QTD TORRE--------------------------



select producao.tb_pg_id, sum(torre) QTD_TORRE from
(select p.tb_pg_id, p.servico,  count (mslink) torre
 from ponto_geografico p where tb_pg_id='11'
 group by p.tb_pg_id, p.servico) producao
 group by tb_pg_id 


----------------------PESQUISAR QTD INSTALACAO0------------

SELECT P.PGF_BARRAMENTO POSTE, P.PGF_UTMX X, P.PGF_UTMY Y FROM INSTALACAO I
INNER JOIN TIPO_DE_INSTALACAO TI ON I.TB_IN_ID=TI.TB_IN_ID
INNER JOIN NO N ON I.NO_ID=N.NO_ID 
INNER JOIN PONTO_GEOGRAFICO P ON N.MSLINK_PG=P.MSLINK

WHERE TI.TB_COD_REAL='BC' AND P.LG_ID=1



-------------------LT PRA QGIS--------------------------


SELECT t.torre_id,TR.MSLINK MSLINK_TRECHO, T.NUMERO_TORRE, T.TB_MATERIAL_ID, LT.LT_NOME_LINHA
,        'LINESTRING(' ||
         ROUND(T.SHAPE.MINX) ||
         ' '||
         ROUND(T.SHAPE.MINY) ||
         ','||
         ROUND(T2.SHAPE.MAXX)||
         ' '||
         ROUND(T2.SHAPE.MAXY)||
         ')' shape
 FROM TRECHO_TRANSMISSAO TR 
inner join no_transmissao n on tr.TT_NO_TRANS_DE_ID=n.NO_TRANS_ID
left join no_transmissao n2 on tr.TT_NO_TRANS_PA_ID=n2.NO_TRANS_ID
left join torre t on n.TORRE_ID=t.TORRE_ID 
left join torre t2 on n2.TORRE_ID=t2.TORRE_ID
INNER JOIN LINHA_TRANSMISSAO LT ON LT.LINHA_TRANS_ID=TR.LINHA_TRANS_ID


 INNER JOIN NO_TRANSMISSAO NO ON NO.NO_TRANS_ID=TR.MSLINK
 INNER JOIN TORRE T ON T.TORRE_ID=NO.TORRE_ID
 INNER JOIN LINHA_TRANSMISSAO LT ON LT.LINHA_TRANS_ID=TR.LINHA_TRANS_ID


------------------------------VERIFICAR FOTOS

SELECT   p.pgf_barramento, p.pgf_utmx,p.pgf_utmy, p.servico,d.arg_id,d.arg_sequencia,d.arg_nome_arquivo,d.arg_data
FROM   digital_arquivo d,elemento_rede_digital e,ponto_coletado_grafico pc,ponto_geografico p,inspecao i
WHERE       d.arg_id = e.arg_id AND e.ponto_coletado_id = pc.ponto_coletado_id AND pc.mslink = p.mslink and p.servico=i.inspecao_nome and pgf_barramento in ('110137233');


-------------------------LOCALIZAR POR TIPO DE POSTE

select p.pgf_barramento, tp.tb_pg_descricao from ponto_geografico p 
inner join tipo_de_pg tp on p.tb_pg_id=tp.tb_pg_id
where tp.tb_pg_descricao='COLETA'      


---------------------CARIMBA POSTE DENTRO DE AMAPA-----------
update ponto_geografico set lg_id ='1' where pgf_utmx >300000 and pgf_utmy >9882000



------------------FOTOS PARA KMZ---------------------------
select 
     p.TB_PG_ID,p.TB_CA_ID,p.PGF_BARRAMento,p.MSLINK,p.PGF_UTMX,p.PGF_UTMY,p.MUN_ID,P.SERVICO,'' numid,'' numfisico,'','' caracteris,'' nox,'' noy,'' FOTOS,pco.longitude,pco.latitude,

   LISTAGG('<img style="max-width:500px;" src="http://sicadhm.progeorn.com.br/download/ceee/FOTOS/' ||e.servico || '/' || d.arg_nome_arquivo || '">' ,'')
    WITHIN GROUP (ORDER BY p.mslink) fotos

    from 
    digital_arquivo d

    inner join elemento_rede_digital e on e.arg_id=d.arg_id
     inner join ponto_coletado_grafico pc on pc.ponto_coletado_id=e.ponto_coletado_id
    inner join ponto_geografico p on pc.mslink=p.mslink
    inner join ponto_coletado pco on pco.ponto_coletado_id=pc.ponto_coletado_id





    where  
    d.arg_sequencia < 10 and
     p.pgf_barramento in ('129323397')
     group by P.SERVICO,p.pgf_barramento,pco.longitude,pco.latitude,p.TB_PG_ID,p.TB_CA_ID,p.PGF_BARRAMento,p.MSLINK,p.PGF_UTMX,p.PGF_UTMY,p.MUN_ID,P.SERVICO

-----------------------------CONTAR POSTE APARTIR DA DATA DE INSTALACAO

select * from ponto_geografico p where p.pgf_data_instalacao > to_date('10/01/2023','dd/mm/yyyy')

-------------------------VERIFICAR POSTE SEM FOTO
SELECT

P.SERVICO
,P.PGF_BARRAMENTO
,pco.ponto_data
,pco.id_pda MSLINK_NO_DB
,pco.tb_elemento_id TIPO
,'\\10.7.1.45\OraFotoDir\CELPA\'|| p.servico foto
,COUNT(d.arg_nome_arquivo) QTD_FOTO

                                           from ponto_geografico P
                                           INNER JOIN  ponto_coletado_grafico pc on pc.mslink=p.mslink
                                           INNER JOIN ponto_coletado pco on pco.ponto_coletado_id=pc.ponto_coletado_id
                                           LEFT JOIN elemento_rede_digital e on pc.ponto_coletado_id=e.ponto_coletado_id
                                           LEFT JOIN digital_arquivo d on e.arg_id=d.arg_id

 


    where  
  --- p.pgf_barramento=('129321323')
p.servico in ('16112022092812143556')  ---and pco.tb_elemento_id in('PG','PE')
GROUP BY P.SERVICO
,P.PGF_BARRAMENTO

,pco.ponto_data
,pco.id_pda 
,pco.tb_elemento_id




---------------------------COORDENADA RIO GRANDE DO SUL----------------
 where P.pgf_UTMY <8000000


------------------------------POSTE POR SUBESTACAO RIO GRANDE DO SUL----------------
select  pgf_barramento,max(round(utmx))utmx,max(round(utmy))utmy,max(se_nome)subestacao  from 
(select distinct p.pgf_barramento,round(p.pgf_utmx)utmx,round(p.pgf_utmy)utmy,se.se_nome from ponto_geografico p
inner join no n on n.mslink_pg=p.mslink
INNER join trecho_de_rede tr on n.no_id=tr.no_id_de or n.no_id=tr.no_id_pa
INNER join trecho_primario tp on tr.mslink=tp.mslink
inner join alimentador al on tp.al_id=al.al_id
inner join barramento_se ba on ba.ba_id=al.ba_id
inner join subestacao se on se.se_id=ba.se_id
where se.se_nome='AGUAS CLARAS'
union all
select distinct p.pgf_barramento,round(p.pgf_utmx)utmx,round(p.pgf_utmy)utmy,se.se_nome 
from ponto_geografico p
inner join no n on n.mslink_pg=p.mslink
INNER join trecho_de_rede tr on n.no_id=tr.no_id_de or n.no_id=tr.no_id_pa
INNER join trecho_secundario ts on tr.mslink=ts.mslink
inner join instalacao i on i.mslink=ts.mslink_ins
inner join alimentador al on i.al_id=al.al_id
inner join barramento_se ba on ba.ba_id=al.ba_id
inner join subestacao se on se.se_id=ba.se_id
where se.se_nome='AGUAS CLARAS')
group by pgf_barramento

-------------------------------------DESCARIMBAR ALIMENTADOR DESMONTADO------------------------------------------
UPDATE TRECHO_PRIMARIO SET AL_ID= null WHERE TRC_ENERGIZADO='N';
COMMIT;


-------------------------------------------------------INSTALACAO POR SUBESTACAO----------------------------------
select  DISTINCT p.pgf_barramento,I.INT_NUM,TI.TB_COD_REAL,round(p.pgf_utmx)utmx,round(p.pgf_utmy)utmy,se.se_nome 
from ponto_geografico p
inner join no n on n.mslink_pg=p.mslink
INNER JOIN INSTALACAO I ON I.NO_ID=n.no_id
INNER JOIN TIPO_DE_INSTALACAO TI ON i.tb_in_id=ti.tb_in_id
inner join alimentador al on i.al_id=al.al_id
inner join barramento_se ba on ba.ba_id=al.ba_id
inner join subestacao se on se.se_id=ba.se_id
where se.se_nome='ALVORARDA 1' AND TI.TB_COD_REAL IN ('CF','FU','CO','CS','CG','SE','RG')


-----------------------------------------TIRAR ERRO NA INSTALAÇÃO INVISIVEL---------------------


UPDATE INSTALACAO SET tb_in_id='13' WHERE MSLINK IN
(
  SELECT 
       I.MSLINK
  FROM INSTALACAO I 
       INNER JOIN NO                  ON NO.NO_ID=I.NO_ID
       INNER JOIN PONTO_gEOGRAFICO PG ON PG.MSLINK=NO.MSLINK_PG
  WHERE 
       PG.PGF_BARRAMENTO = '129509956'
)

