# Ingress SNI Passthrough

## Simple TLS Mode

1. Create a secret `ngnix-certificate` where the istio-ingressgateway is running
    ```
    openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout example.com.key -out example.com.crt
    openssl req -out nginx.example.com.csr -newkey rsa:2048 -nodes -keyout nginx.example.com.key -subj "/CN=nginx.example.com/O=some organization"
    openssl x509 -req -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 0 -in nginx.example.com.csr -out nginx.example.com.crt

    export CONTROL_PLANE=istio-system

    oc create secret tls nginx-certificate --key nginx.example.com.key --cert nginx.example.com.crt -n $CONTROL_PLANE
    ```

2. Launch the deployment
    ```
    export DATA_PLANE=app-namespace

    export BASE_DOMAIN=apps.$(oc get dns cluster -o jsonpath='{.spec.baseDomain}')

    helm upgrade --install app nginx/ -f cd/simple.yaml --set mesh.baseDomain=$BASE_DOMAIN -n $DATA_PLANE
    ```

3. Test the gateway

    ```
    export GATEWAY_URL=$(oc get route -lmaistra.io/gateway-name=app-mesh -o jsonpath='{.items..spec.host}' -n $CONTROL_PLANE)
    curl -k https://$GATEWAY_URL/health
    ```

## Passthrough Mode

2. Launch the deployment
    ```
    export DATA_PLANE=app-namespace

    export BASE_DOMAIN=apps.$(oc get dns cluster -o jsonpath='{.spec.baseDomain}')

    helm upgrade --install app nginx/ -f cd/passthrough.yaml --set mesh.baseDomain=$BASE_DOMAIN -n $DATA_PLANE
    ```
    >
    > It using the `service.alpha.openshift.io/serving-cert-secret-name`annotations for the certificate.
    > 
    
3. Test the gateway

    ```
    export GATEWAY_URL=$(oc get route -lmaistra.io/gateway-name=app-mesh -o jsonpath='{.items..spec.host}' -n $CONTROL_PLANE)
    curl -k https://$GATEWAY_URL/health
    ```
