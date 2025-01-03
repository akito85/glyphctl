You are allowed to use env variable in PostgreSQL Docker as PGPASSWORD to avoid prompt password when executing the script

```bash

#!/bin/bash

wget --recursive --no-parent http://192.168.200.210/dbsisfo/kostrad/ -P /home/kostrad/updatedb

cat DB_PERS_POKOK.csv | PGPASSWORD=psql16 psql -h 127.0.0.1 -U postgres -p 5555 -d comcent -c "TRUNCATE TABLE api.db_pers_pokok RESTART IDENTITY;COPY api.db_pers_pokok (nopers, notam, nm_pers, gelardpn, gelarblk, kd_pkt, tmt_pkt, kd_corps, kd_ktm, kd_smkl, kd_ktgr, tmt_ktgr, kd_jenjab, kd_ktgr_jab, kd_esjab, no_jab_skep, ur_jab_skep, tmt_jab_skep, kd_pje, tmt_tni, tmt_pa, kd_dikma, th_llsdikma, kd_dikum, th_llsdikum, kd_jns_kelamin, tgl_lahir, kd_prop, kd_kab, kd_kec, kd_desa, kd_agama, kd_stapers, kd_suku, uktinggi, ukberat, kd_intel, kd_kulit, kd_btk_rambut, kd_wrn_rambut, kd_darah, kd_stakel, jml_anaks, no_kpi_kps, kd_starumgal, alamat, kd_prop_alamat, kd_kab_alamat, kd_kec_alamat, kd_desa_alamat, tlp, email, no_kk, nik, npwp, asabri, id_bpjs, ppk, grade, update_time, update_user, update_status, tmt_jab, kd_dikmil, th_lls, thlls_dikum, kd_prop_lahir, kd_kab_lahir, kd_kec_lahir, kd_desa_lahir) FROM STDIN WITH DELIMITER ',' CSV"

# or via podman

cat /home/kostrad/updatedb/192.168.200.210/dbsisfo/kostrad/DB_PERS_POKOK.csv | podman exec -i db-postgresql /bin/bash -c "PGPASSWORD=pgsql psql -h 192.168.20.9 -U user -p 5555 -d comcent -c \"TRUNCATE TABLE api.db_pers_pokok RESTART IDENTITY;COPY api.db_pers_pokok (nopers, notam, nm_pers, gelardpn, gelarblk, kd_pkt, tmt_pkt, kd_corps, kd_ktm, kd_smkl, kd_ktgr, tmt_ktgr, kd_jenjab, kd_ktgr_jab, kd_esjab, no_jab_skep, ur_jab_skep, tmt_jab_skep, kd_pje, tmt_tni, tmt_pa, kd_dikma, th_llsdikma, kd_dikum, th_llsdikum, kd_jns_kelamin, tgl_lahir, kd_prop, kd_kab, kd_kec, kd_desa, kd_agama, kd_stapers, kd_suku, uktinggi, ukberat, kd_intel, kd_kulit, kd_btk_rambut, kd_wrn_rambut, kd_darah, kd_stakel, jml_anaks, no_kpi_kps, kd_starumgal, alamat, kd_prop_alamat, kd_kab_alamat, kd_kec_alamat, kd_desa_alamat, tlp, email, no_kk, nik, npwp, asabri, id_bpjs, ppk, grade, update_time, update_user, update_status, tmt_jab, kd_dikmil, th_lls, thlls_dikum, kd_prop_lahir, kd_kab_lahir, kd_kec_lahir, kd_desa_lahir) FROM STDIN WITH DELIMITER ',' CSV\""
```
