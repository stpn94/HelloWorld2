SELECT *
  FROM (SELECT VEA.EMPNO,
               REPLACE(VEA.KORN_FLNM, ' ', '') AS KORN_FLNM,
               TAM.ORGCD,
               TAM.JBGD_NM,
               TAM.JBPS_NM,
               TAM.DUTY_NM,
               TAM.SDTS_CRTR_YMD,
               TAM.DUTY_CD,
               TAM.JBGD_CD,
               TAM.JBPS_CD
          FROM DB_EM.VW_EMP_ALL VEA
               JOIN (SELECT EMPNO,
               				A.ROW_NUM,
                            APTMT_YMD,
                            ORGCD,
                            DUTY_CD,
                            JBGD_CD,
                            JBPS_CD,
                            JBGD_NM,
                            JBPS_NM,
                            DUTY_NM,
                            SDTS_CRTR_YMD
                       FROM (SELECT ROW_NUMBER() OVER(PARTITION BY TAM.EMPNO ORDER BY TAM.EMPNO, TAM.APTMT_YMD DESC) AS ROW_NUM, --첫번째 발령정보 조회용
                                    TAM.*
                               FROM DB_EM.TB_APTMT_MTTR TAM
                              WHERE TAM.APTMT_YMD <= '20230101' --발령일
                                AND NOT APTMT_CD IN ( '00D5', '00D9', '00B5', '00D2',
                                                      '00D6', '00D0', '00D3', '00AC', '00J2' ) -- 직위,직급,직렬 안 바뀌는 발령 코드
                            ) A
                      WHERE ROW_NUM = 1) TAM
                 ON TAM.EMPNO = VEA.EMPNO	
                    AND ( VEA.RSGNTN_YMD IS NULL
                           OR VEA.RSGNTN_YMD > '20230101' ) -- 퇴사자 제외
       ) A
       LEFT JOIN DB_EM.VW_BSCD
              ON A.ORGCD = VW_BSCD.DEPT_CD
                 AND '20230101' BETWEEN VW_BSCD.USE_BGNG_YMD AND VW_BSCD.USE_END_YMD
 WHERE DEPTMT_ORGCD = '1101007'
 ORDER BY SEQ,
          DUTY_CD,
          JBGD_CD,
          JBPS_CD,
          EMPNO ;
