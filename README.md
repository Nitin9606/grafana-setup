# grafana-setup

# Define the content for the notes
notes_content = """
Step-by-Step: Set Up Grafana with Loki and Promtail on EC2 using Docker

1. Launch EC2 Instance
- Use Ubuntu as the base image.
- Allow ports 3000, 3100, and 22 in the Inbound Rules of the EC2 Security Group.

2. Install Grafana (Stable Release)
sudo apt-get update
sudo apt-get install -y grafana
sudo systemctl start grafana
sudo systemctl enable grafana
- Access Grafana at http://<EC2-IP>:3000/ (default login: admin/admin)

3. Install Docker
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
sudo reboot

4. Create Directory for Loki and Promtail Config
mkdir grafana
cd grafana
wget https://raw.githubusercontent.com/grafana/loki/main/cmd/loki/loki-local-config.yaml -O loki-config.yaml
wget https://raw.githubusercontent.com/grafana/loki/main/clients/cmd/promtail/promtail-local-config.yaml -O promtail-config.yaml

5. Run Loki Container
docker run -d --name=loki \\
  -p 3100:3100 \\
  -v $(pwd)/loki-config.yaml:/etc/loki/local-config.yaml \\
  grafana/loki:2.9.3 \\
  -config.file=/etc/loki/local-config.yaml

Check endpoints:
- http://<EC2-IP>:3100/ready
- http://<EC2-IP>:3100/metrics

6. Run Promtail Container (Linked with Loki)
docker run -d --name=promtail \\
  -v /var/log:/var/log \\
  -v $(pwd)/promtail-config.yaml:/etc/promtail/promtail-config.yaml \\
  grafana/promtail:2.9.3 \\
  -config.file=/etc/promtail/promtail-config.yaml

7. Configure Grafana
- Go to Grafana dashboard → Add data source
- Select Loki
- URL: http://localhost:3100
- Click Save & Test

8. Check Logs in Grafana
- Go to Explore
- In Label filters, select job and then varlogs
- Run queries (e.g., line contains "nginx")

9. Add Custom Log Source (e.g., Grafana Logs)
Edit promtail config:
- job_name: grafanalogs
  static_configs:
    - targets:
        - localhost
      labels:
        job: grafanalogs
        __path__: /var/log/grafana/*log

Save config and restart promtail:
docker restart promtail

10. (Optional) Generate Logs with Nginx
sudo apt-get install nginx
Visit http://<EC2-IP> to generate access logs.

In Grafana → Explore:
- Filter: job = varlogs
- Query: line contains "nginx"
- Select last 5 minutes
- Add visualisation with functions like rate() or count_over_time()
"""

# Define file path
file_path = Path("/mnt/data/Grafana_Loki_Promtail_Setup_Notes.txt")

# Write to file
file_path.write_text(notes_content)

file_path
