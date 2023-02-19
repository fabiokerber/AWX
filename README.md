Credits:<br>

http://vcloud-lab.com/entries/devops/how-to-install-ansible-awx-on-ubuntu-using-kubernetes-k8s<br>
https://github.com/kubernetes-sigs/kustomize<br>
https://github.com/ansible/awx<br>
https://github.com/ansible/awx-operator<br>

`$ sudo kubectl get all -n awx`<br>
`$ sudo kubectl get service awx-local-service -n awx`<br>
`$ sudo kubectl get secret awx-local-admin-password -o jsonpath="{.data.password}" -n awx | base64 --decode ; echo`<br>

http://192.168.56.10:30080<br>
admin<br>
pass<br>