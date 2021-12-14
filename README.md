# Service Mesh Upgrade
## Prerequisite
- Have openshift Service Mesh v1.1 installed

## Install Bookinfo application
- create a project
Command: oc new-project bookinfo
- click on installed operators
- go to the project where you have installed service mesh
- click on service mesh -> click on Istio Service Mesh Member Roll
- add bookinfo project in spec.members
- or you could patch it with following command
  Command: oc -n <control plane project> patch --type='json' smmr default -p '[{"op": "add", "path": "/spec/members", "value":["'"bookinfo"'"]}]'
- install the application using the following command
  Command: oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/istio/maistra-1.1/samples/bookinfo/platform/kube/bookinfo.yaml
- install gateway 
  Command: oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/istio/maistra-1.1/samples/bookinfo/networking/bookinfo-gateway.yaml
- add destination rule
  Command: oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/istio/maistra-1.1/samples/bookinfo/networking/destination-rule-all.yaml
- add mutual tls
  Command: oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/istio/maistra-1.1/samples/bookinfo/networking/destination-rule-all-mtls.yaml

  
Go to your project on web console, click on routes and then click on ingress-gateway.
Now add '/productpage' at the end of url and you should see your application.
  
## Upgrade Openshift Service Mesh
- Check your v1 smcp and make sure it's valid
  Command: oc get smcp -o yaml ( inside your service mesh application)
- if there are any invalid fields, they will appear in spec.techPreview.errored.message.
- After fixing them replace the smcp
  Command: oc replace -f <name of smcp>.yaml
  
- Backup your current smcp, by going to the project with current control plane
- get the control plane as v1
  command: oc get servicemeshcontrolplanes.v1.maistra.io <smcp_name> -o yaml > <smcp_name>.v1.yaml
- convert it v2
  command: oc get smcp <smcp_name> -o yaml > <new smcp name>.v2.yaml
- now, change the namespace inside metadata to new project where you will create smcp v2.0
- also change version from v1.1 to v2.0
  
- Create new project
  Command: oc new-project <sm project name> -f <new smcp name>.v2.yaml

- Delete bookinfo from original service mesh member roll, and add it to the new one.
- Go to new project -> networking -> routes
- Delete the ingress-gateway route that shows a red icon and says not accepted

  
## Check if your new version is working
- in routes of the new project, click on on the link for ingress gatway and then add /productpage at end..if it works your installation was successful.
