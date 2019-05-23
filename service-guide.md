## Creazione di un servizio che si avvia automaticamente in Ubuntu 18.04



1. Creare uno script da eseguire (e.g. /home/guido/my_script.sh)

2. Aggiungere al file i permessi di esecuzione `chmod +x /home/guido/my_script.sh`

3. Aggiungere la definizione del servizio

    - Creare un file /usr/local/bin/my_service.sh

    - Aggiungere al file i permessi di esecuzione `chmod +x /usr/local/bin/my_service.sh`

    - Utilizzare questo template sostituendo opportunamente il percorso del pidfile con uno a piacimento, idem per le echo "My Service" e per il percorso dello script da eseguire.

      ```bash
      #!/bin/bash
      
      pidfile="/tmp/my_service.pid"
      
      function d_start()
      {
              echo "My Service: starting service"
              bash /home/guido/my_script.sh & echo $! > $pidfile
              echo "PID is $(cat $pidfile)"
      }
      function d_stop()
      {
              echo "My Service: stopping Service (PID=$(cat $pidfile))"
              kill $(cat $pidfile)
              rm $pidfile
       }
      function d_status ()
      {
              ps -ef | grep my_service | grep -v grep
              echo "PID indicate indication file $(cat $pidfile 2&gt; /dev/null)"
      }
      # Management instructions of the service
      case "$1" in
              start)
                      d_start
                      ;;
              stop)
                      d_stop
                      ;;
              reload)
                      d_stop
                      sleep 1
                      d_start
                      ;;
              status)
                      d_status
                      ;;
              *)
              echo "Usage: $0{start|stop|reload|status}"
              exit 1
              ;;
      esac
      exit 0
      ```

4. Aggiungere la configurazione del servizio

    - Creare il file /etc/systemd/system/my_service.service

    - Utilizzare questo template sostituendo oppportunamente i valori

      ```bash
      [Unit]
      Description=My Service
      After=network.target 
      
      [Service]
      User=www-data
      Group=www-data
      Type=forking
      Restart=on-failure
      ExecStart = /usr/local/bin/my_service.sh start
      ExecStop = /usr/local/bin/my_service.sh stop
      ExecReload = /usr/local/bin/my_service.sh reload
      
      [Install]
      WantedBy=multi-user.target
      ```

5. Riavviare il gestore dei servizi (systemctl)

   `sudo systemctl daemon-reload`

6. Per aggiungerlo ai servizi attivi all'avvio del computer utilizzare

   `sudo systemctl enable my_service.service`

7. Testare che tutto funzioni, innanzitutto verifichiamo che il servizio esista

   `sudo service my_service status`

8. Quindi provare ad avviarlo

   `sudo service my_service start`

9. Se siamo soddisfatti non ci resta che provare a riavviare il server per verificare che il servizio si attivi automaticamente (possiamo verificare che tutto sia andato a buon fine con il comando del punto 7).

10. Per interrompere manualmente l'esecuzione del servizio possiamo utilizzare il comando stop

    `sudo service my_service stop`