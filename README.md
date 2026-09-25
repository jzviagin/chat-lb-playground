# Chat LB playground

HAProxy (leastconn) in front of two FastAPI WebSocket echo instances.

## Run locally

    docker compose up --build

- Stats page: http://localhost:8404
- Open several clients: `npx wscat -c ws://localhost/ws`
  (each greets you with the instance it landed on)
- Failover: `docker compose stop chat1` -> its sockets drop, reconnects land on chat2
- Recovery: `docker compose start chat1` -> new connections prefer chat1 (fewest sockets)

## Run on an Oracle Cloud Always Free VM

1. Create instance: Ubuntu 24.04, shape VM.Standard.A1.Flex (e.g. 2 OCPU / 12 GB), public IP, your SSH key.
2. VCN -> subnet -> Security List: ingress TCP 80 and 443 from 0.0.0.0/0 (do NOT open 8404).
3. On the VM, open the OS firewall (Oracle images reject everything but SSH):

       sudo iptables -I INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT
       sudo netfilter-persistent save

4. Install Docker:

       curl -fsSL https://get.docker.com | sudo sh
       sudo usermod -aG docker $USER   # then log out/in

5. Copy this folder over (`scp -r chat-lb-playground ubuntu@<IP>:`) and run `docker compose up -d --build`.
6. Test: `npx wscat -c ws://<IP>/ws`
   Stats via SSH tunnel: `ssh -L 8404:localhost:8404 ubuntu@<IP>` then open http://localhost:8404
