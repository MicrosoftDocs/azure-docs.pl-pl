---
title: Używanie niestandardowego kontrolera ruchu przychodzących NGINX i konfigurowanie protokołu HTTPS
services: azure-dev-spaces
ms.date: 12/10/2019
ms.topic: conceptual
description: Dowiedz się, jak skonfigurować Azure Dev Spaces do korzystania z niestandardowego kontrolera ruchu przychodzących NGINX i skonfigurować protokół HTTPS przy użyciu tego kontrolera ruchu przychodzących
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, containers, Helm, service mesh, service mesh routing, kubectl, k8s
ms.custom: devx-track-js
ms.openlocfilehash: b07d66e0031b907811c4ec251987aa020542b05a
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777513"
---
# <a name="use-a-custom-nginx-ingress-controller-and-configure-https"></a>Używanie niestandardowego kontrolera ruchu przychodzących NGINX i konfigurowanie protokołu HTTPS

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

W tym artykule pokazano, jak skonfigurować Azure Dev Spaces do korzystania z niestandardowego kontrolera ruchu przychodzących NGINX. W tym artykule pokazano również, jak skonfigurować niestandardowy kontroler ruchu przychodzących do korzystania z protokołu HTTPS.

## <a name="prerequisites"></a>Wymagania wstępne

* Subskrypcja platformy Azure. Jeśli nie masz, możesz utworzyć [bezpłatne konto][azure-account-create].
* [Zainstalowany interfejs wiersza polecenia platformy Azure][az-cli].
* Azure Kubernetes Service (AKS) z włączonym Azure Dev Spaces .
* [Zainstalowane oprogramowanie kubectl.][kubectl]
* [Program Helm 3 zainstalował program][helm-installed].
* [Domena niestandardowa ze][custom-domain] [strefą DNS.][dns-zone]  W tym artykule przyjęto założenie, że domena niestandardowa i strefa DNS znajdują się w tej samej grupie zasobów co klaster usługi AKS, ale istnieje możliwość użycia domeny niestandardowej i strefy DNS w innej grupie zasobów.

## <a name="configure-a-custom-nginx-ingress-controller"></a>Konfigurowanie niestandardowego kontrolera ruchu przychodzących NGINX

Połącz się z klastrem przy użyciu [narzędzia kubectl][kubectl], klienta wiersza polecenia kubernetes. Aby skonfigurować narzędzie `kubectl` w celu nawiązania połączenia z klastrem Kubernetes, użyj polecenia [az aks get-credentials][az-aks-get-credentials]. To polecenie powoduje pobranie poświadczeń i zastosowanie ich w konfiguracji interfejsu wiersza polecenia Kubernetes.

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKS
```

Aby sprawdzić połączenie z klastrem, użyj polecenia [kubectl get][kubectl-get], aby powrócić do listy węzłów klastra.

```console
kubectl get nodes
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.14.1
```

Dodaj oficjalne [stabilne repozytorium Helm,][helm-stable-repo]które zawiera wykres Helm kontrolera ruchu przychodzących NGINX.

```console
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

Utwórz przestrzeń nazw Kubernetes dla kontrolera ruchu przychodzących NGINX i zainstaluj ją przy użyciu elementu `helm` .

```console
kubectl create ns nginx
helm install nginx stable/nginx-ingress --namespace nginx --version 1.27.0
```

> [!NOTE]
> Powyższy przykład tworzy publiczny punkt końcowy dla kontrolera ruchu przychodzących. Jeśli zamiast tego musisz użyć prywatnego punktu końcowego dla kontrolera ruchu przychodzących, dodaj *parametr --set controller.service.annotations". service \\ .beta \\ .kubernetes \\ .io/azure-load-balancer-internal"=true* parametr polecenia *helm install.* Na przykład:
> ```console
> helm install nginx stable/nginx-ingress --namespace nginx --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"=true --version 1.27.0
> ```
> Ten prywatny punkt końcowy jest ujawniony w sieci wirtualnej, w której wdrożono klaster usługi AKS.

Pobierz adres IP usługi kontrolera ruchu przychodzących NGINX przy użyciu [narzędzia kubectl get][kubectl-get].

```console
kubectl get svc -n nginx --watch
```

Przykładowe dane wyjściowe pokazują adresy IP wszystkich usług w *przestrzeni nazw serwera nginx.*

```console
NAME                                  TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
nginx-nginx-ingress-controller        LoadBalancer   10.0.19.39     <pending>        80:31314/TCP,443:30521/TCP   10s
nginx-nginx-ingress-default-backend   ClusterIP      10.0.210.231   <none>           80/TCP                       10s
...
nginx-nginx-ingress-controller        LoadBalancer   10.0.19.39     MY_EXTERNAL_IP   80:31314/TCP,443:30521/TCP   26s
```

Dodaj rekord *A* do strefy DNS przy użyciu zewnętrznego adresu IP usługi NGINX za pomocą [narzędzia az network dns record-set a add-record][az-network-dns-record-set-a-add-record].

```azurecli
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name *.nginx \
    --ipv4-address MY_EXTERNAL_IP
```

W powyższym przykładzie dodano *rekord A* do *MY_CUSTOM_DOMAIN* DNS.

W tym artykule użyjemy przykładowej [aplikacji Azure Dev Spaces Bike Sharing,](https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp) aby zademonstrować korzystanie z Azure Dev Spaces. Klonowanie aplikacji z usługi GitHub i przechodzenie do jej katalogu:

```cmd
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/BikeSharingApp/charts
```

Otwórz [program values.yaml][values-yaml] i dokonaj następujących aktualizacji:
* Zastąp wszystkie wystąpienia<REPLACE_ME_WITH_HOST_SUFFIX>przy *użyciu* *serwera nginx. MY_CUSTOM_DOMAIN* domeny na MY_CUSTOM_DOMAIN *.* 
* Zastąp *kubernetes.io/ingress.class: traefik-azds # Dev Spaces specyficzny* dla *kubernetes.io/ingress.class: nginx # Niestandardowy ruch przychodzący.* 

Poniżej znajduje się przykład zaktualizowanego `values.yaml` pliku:

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
    hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
    hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
```

Zapisz zmiany i zamknij plik.

Utwórz obszar *dewelopera* z przykładową aplikacją przy użyciu metody `azds space select` .

```console
azds space select -n dev -y
```

Wd wdrażaj przykładową aplikację przy użyciu narzędzia `helm install` .

```console
helm install bikesharingsampleapp . --dependency-update --namespace dev --atomic
```

W powyższym przykładzie przykładowa aplikacja jest wdrażana w przestrzeni *nazw dev.*

Wyświetl adresy URL, aby uzyskać dostęp do przykładowej aplikacji przy `azds list-uris` użyciu .

```console
azds list-uris
```

Poniższe dane wyjściowe pokazują przykładowe adresy URL z tabeli `azds list-uris` .

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/  Available
http://dev.gateway.nginx.MY_CUSTOM_DOMAIN/         Available
```

Przejdź do usługi *bikesharingweb,* otwierając publiczny adres URL z `azds list-uris` polecenia . W powyższym przykładzie publiczny adres URL usługi *bikesharingweb* to `http://dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/` .

> [!NOTE]
> Jeśli zamiast usługi *bikesharingweb* zostanie wyświetlony komunikat o  błędzie, sprawdź, czy zaktualizowano zarówno adnotację kubernetes.io/ingress.class, jak i hosta w pliku *values.yaml.* 

Użyj polecenia , aby utworzyć przestrzeń podrzędną w obszarze dev i wyświetlić listę adresów URL w `azds space select` celu uzyskania dostępu do podrzędnej przestrzeni dewelopera. 

```console
azds space select -n dev/azureuser1 -y
azds list-uris
```

Poniższe dane wyjściowe pokazują przykładowe adresy URL z , aby uzyskać dostęp do `azds list-uris` przykładowej aplikacji w podrzędnej przestrzeni dewelopera *azureuser1.*

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/  Available
http://azureuser1.s.dev.gateway.nginx.MY_CUSTOM_DOMAIN/         Available
```

Przejdź do usługi *bikesharingweb* w podrzędnej przestrzeni *deweloperskiej azureuser1,* otwierając publiczny adres URL z `azds list-uris` polecenia . W powyższym przykładzie publiczny adres URL usługi *bikesharingweb* w podrzędnej przestrzeni deweloperskiej *azureuser1* to `http://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/` .

## <a name="configure-the-nginx-ingress-controller-to-use-https"></a>Konfigurowanie kontrolera ruchu przychodzących NGINX do korzystania z protokołu HTTPS

Użyj [programu cert-manager,][cert-manager] aby zautomatyzować zarządzanie certyfikatem protokołu TLS podczas konfigurowania kontrolera ruchu przychodzących NGINX do korzystania z protokołu HTTPS. Użyj `helm` , aby zainstalować pakiet *certmanager.*

```console
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml --namespace nginx
kubectl label namespace nginx certmanager.k8s.io/disable-validation=true
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager --namespace nginx --version v0.12.0 jetstack/cert-manager --set ingressShim.defaultIssuerName=letsencrypt --set ingressShim.defaultIssuerKind=ClusterIssuer
```

Utwórz plik `letsencrypt-clusterissuer.yaml` i zaktualizuj pole adresu e-mail przy użyciu swojego adresu e-mail.

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: MY_EMAIL_ADDRESS
    privateKeySecretRef:
      name: letsencrypt
    solvers:
      - http01:
          ingress:
            class: nginx
```

> [!NOTE]
> Na użytek testowania istnieje również [serwer przejściowy,][letsencrypt-staging-issuer] który może być serwerem *klastra.*

Użyj `kubectl` , aby zastosować `letsencrypt-clusterissuer.yaml` .

```console
kubectl apply -f letsencrypt-clusterissuer.yaml --namespace nginx
```

Zaktualizuj [plik values.yaml,][values-yaml] aby uwzględnić szczegóły dotyczące korzystania z *narzędzia cert-manager i* protokołu HTTPS. Poniżej znajduje się przykład zaktualizowanego `values.yaml` pliku:

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN
      secretName: dev-bikesharingweb-secret

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN
      secretName: dev-gateway-secret
```

Uaktualnij przykładową aplikację przy użyciu narzędzia `helm` :

```console
helm upgrade bikesharingsampleapp . --namespace dev --atomic
```

Przejdź do przykładowej aplikacji w przestrzeni podrzędnej *dev/azureuser1* i zwróć uwagę, że nastąpi przekierowanie do korzystania z protokołu HTTPS. Zauważ również, że strona ładuje się, ale przeglądarka wyświetla pewne błędy. Otwarcie konsoli przeglądarki pokazuje błąd związany ze stroną HTTPS próbującą załadować zasoby HTTP. Na przykład:

```console
Mixed Content: The page at 'https://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/devsignin' was loaded over HTTPS, but requested an insecure resource 'http://azureuser1.s.dev.gateway.nginx.MY_CUSTOM_DOMAIN/api/user/allUsers'. This request has been blocked; the content must be served over HTTPS.
```

Aby naprawić ten błąd, zaktualizuj [wartość BikeSharingWeb/azds.yaml][azds-yaml] podobną do poniższej:

```yaml
...
    ingress:
      annotations:
        kubernetes.io/ingress.class: nginx
        cert-manager.io/cluster-issuer: letsencrypt
      hosts:
      # This expands to [space.s.][rootSpace.]bikesharingweb.<random suffix>.<region>.azds.io
      - $(spacePrefix)$(rootSpacePrefix)bikesharingweb.nginx.MY_CUSTOM_DOMAIN
      tls:
      - hosts:
        - $(spacePrefix)$(rootSpacePrefix)bikesharingweb.nginx.MY_CUSTOM_DOMAIN
        secretName: dev-bikesharingweb-secret
...
```

Zaktualizuj [właściwość BikeSharingWeb/package.jsna przy][package-json] użyciu zależności pakietu *adresu* URL.

```json
{
...
    "react-responsive": "^6.0.1",
    "universal-cookie": "^3.0.7",
    "url": "0.11.0"
  },
...
```

Zaktualizuj metodę *getApiHostAsync* w [pliku BikeSharingWeb/lib/helpers.js,][helpers-js] aby używać protokołu HTTPS:

```javascript
...
    getApiHostAsync: async function() {
        const apiRequest = await fetch('/api/host');
        const data = await apiRequest.json();
        
        var urlapi = require('url');
        var url = urlapi.parse(data.apiHost);

        console.log('apiHost: ' + "https://"+url.host);
        return "https://"+url.host;
    },
...
```

Przejdź do katalogu `BikeSharingWeb` i użyj funkcji , aby uruchomić `azds up` zaktualizowaną *usługę BikeSharingWeb.*

```console
cd ../BikeSharingWeb/
azds up
```

Przejdź do przykładowej aplikacji w przestrzeni podrzędnej *dev/azureuser1* i zwróć uwagę, że nastąpi przekierowanie do używania protokołu HTTPS bez żadnych błędów.

## <a name="next-steps"></a>Następne kroki

Dowiedz się więcej o tym, Azure Dev Spaces działa.

> [!div class="nextstepaction"]
> [Jak działa usługa Azure Dev Spaces](../how-dev-spaces-works.md)


[az-cli]: /cli/azure/install-azure-cli
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-network-dns-record-set-a-add-record]: /cli/azure/network/dns/record-set/a#az_network_dns_record_set_a_add_record
[custom-domain]: ../../app-service/manage-custom-dns-buy-domain.md#buy-an-app-service-domain
[dns-zone]: ../../dns/dns-getstarted-cli.md
[azds-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/azds.yaml
[azure-account-create]: https://azure.microsoft.com/free
[cert-manager]: https://cert-manager.io/
[helm-installed]: https://helm.sh/docs/intro/install/
[helm-stable-repo]: https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository
[helpers-js]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/lib/helpers.js#L7
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[letsencrypt-staging-issuer]: https://cert-manager.io/docs/configuration/acme/#creating-a-basic-acme-issuer
[package-json]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/package.json
[values-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/charts/values.yaml
