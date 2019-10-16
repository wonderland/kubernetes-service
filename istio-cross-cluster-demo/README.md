# Istio Cross-Cluster Demo

# Table of Contents
1. Tools & Resources
1. Steps 
    1. [Create istio Cross-Cluster Mesh](#steps)
    1. [Prep Cluster A](#prep-cluster-a)
        1. [Create the `DestinationRule`](#create-the-destinationrule)
        1. [Route traffic to Cluster B](#route-traffic-to-cluster-b)
    1. [Prep Cluster B](#prep-cluster-b)
    1. [See the Cross-Cluster Action](#see-the-cross-cluster-action)
1. [Special thanks](#props)
    

# Tools & Resources
1. Stackpoint.io or [NetApp Kubernetes Service (NKS)](netapp.stackpoint.io)
2. Istio 1.0 on 2 separate K8s clusters, [install via Stackpoint.io web app](https://stackpoint.io/clusters/new?solution=istio). 
3. The `.yaml` files included in this repo. 
4. latest version of `kubectl`
, [installation instructions](https://istio.io/docs/setup/kubernetes/quick-start/#download-and-prepare-for-the-installation)
5. [newcluster.sh](newcluster.sh) to quickly enable **kubeconfig**

# Steps

## Create Istio Cross-Cluster Mesh

1. Go to the Istio Mesh tab on [Stackpoint.io](https://stackpoint.io/istio-meshes/list) or [NKS](https://netapp.stackpoint.io/istio-meshes/list).
2. Press the Create Istio Mesh button
3. Complete the 3-step wizard

## Prep Cluster A
1. Select one of the clusters in the mesh to go to the detail page. We'll refer to this as Cluster A moving forward. 
2. Download and enable **kubeconfig** file from the cluster's detail page. 
3. `sh newcluster.sh`, or `export KUBECONFIG=/path/to/kubeconfig`
4. Deploy _bookinfo_ using the attached manifest, `kubectl apply -f cluster-a-bookinfo-manifests.yaml`
5. Identify the host info for Cluster B by reading the egress rule on Cluster A, `kubectl describe serviceentry spccc-egress-service-entry`

Look for this:

    Spec:
      Endpoints:
        Address:  35.199.144.87
      Hosts:
        svc.netpgcxz4x.remote


### Create the `DestinationRule`

6. Add the host info for cluster B to the `destinationrule.yaml` so that cluster A knows where to send traffic.
7. Apply the rule to Cluster A, `kubectl apply -f destinationrule.yaml`

### Route traffic to Cluster B

8. Add the host info for cluster B to the `splitter.yaml` files; it needs to be added in two places. 

```
  http:
  - rewrite:
      authority: reviews.default.svc.netpgcxz4x.remote
    route:
    - destination:
        host: svc.netpgcxz4x.remote
```

9. Save your changes. 
10. Apply the update to Cluster A, `kubectl apply -f splitter.yaml`

## Prep Cluster B

1. Go to the detail page for the other cluster in the cross-cluster mesh, Cluster B. 
2. Download and enable **kubeconfig** file from the cluster's detail page. 
3. `sh newcluster.sh`, or `export KUBECONFIG=/path/to/kubeconfig`
4. Deploy _bookinfo_ using the attached manifest, `kubectl apply -f cluster-b-bookinfo-manifests.yaml`

## See the Cross-Cluster Action

1. On the detail page, scroll down to the Solutions section.  
2. Select the Istio ingress URL 
3. Appended `/productpage` to the link, e.g. http://35.227.159.142/productpage
4. Refresh the page. 
5. Refresh the page. 
6. Refresh the page. 

You should see both "no stars" and "red stars" reviews. This indicates that traffic is being split between the clusters!

# Props
Special thanks to @justinhopper and @erikabarcott for their help on this demo!

