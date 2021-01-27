# APIChaos Experiment
This experiment allows you to inject chaos into an API, and observe the effect on its dependencies and even on the whole cluster.  In this experiment, you can control the latency, success rate, and return code of an API’s response.

## Prerequest
To observe the effect on an API and its dependencies, you need to set up some monitoring dashboard and alert info in your cluster. For a better experience, to observe the effect on a calling chain when injecting API chaos, strongly suggest you install Skywalking. DO NOT have one? Follow [Skywalking](https://github.com/apache/skywalking/blob/8.3.0/docs-hotfix/docs/en/setup/README.md) and [SkyWalkingSample](https://github.com/beckjin/SkyWalkingSample) to install.

The following will use Skywalking to show the experiment steps and how to verify the result.

## API latency configuration file
### Example file
```
# cat api-latency.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: APIChaos
metadata:
  name: api-latency
  namespace: chaos-testing
spec:
  action: api-latency
  duration: '30m'
  responsedelay: '30ms'
  percentage: 100%
  selector:
    labelSelectors:
      'app.kubernetes.io/component': 'projectA'
  port: 8761
  endpoint: /projectA/test
```

In this file, the experiment will inject 30ms latency for 30 minutes, on the projectA pod's 8761 port when calling `/projectA/test` API. Below are details of each parameter.

### Parameters
* **action** `api-latency` means inject delay to an API. `api-failure` means inject error response to an API. In this sample, it injects latency to an API.
* **duration** defines the duration for each chaos experiment. In the sample file above, the time chaos lasts for 30 minutes.
* **responsedelay** defines the response delay time when action is `api-latency`. In the sample above, it's 30ms.
* **responsecode** defines the return code when action is `api-failure`.
*  **responsebody** defines the http response body.
* **percentage** defines how much API response will be injected by the chaos experiment. 100% means all API responses will be injected, 50% means only half of the API response will be injected.
* **selector** specifies the target pods for chaos injection. For more details, see Define the Scope of Chaos Experiment.
* **external** instead of injecting chaos to internal service, define an external domain to simulate external API failure. For example, `www.baidu.com`.
* **port** defines the target port of selected pods.
* **endpoint** defines the target API endpoint of the selected pods.

### Steps to inject failure and verify the result
1. Apply api-latency.yaml
```
# kubectl apply -f api-latency.yaml
```
2. Open Skywalking and wait for 30 minutes, you will observe below changes in the dashboard.
    The overall dependency relationship of the sample applications. Will inject API chaos on projectA API, and observe changes on projectB and projectC pods.
![Screen Shot 2021-01-27 at 3 49 17 PM](https://user-images.githubusercontent.com/24226678/105968448-11979200-60c2-11eb-8581-f4138f696a05.png)

    Observe metrics changes.
![Screen Shot 2021-01-27 at 3 57 12 PM](https://user-images.githubusercontent.com/24226678/105968479-19573680-60c2-11eb-875f-95fe2a0dfb90.png)
![Screen Shot 2021-01-27 at 3 57 31 PM](https://user-images.githubusercontent.com/24226678/105968486-1b20fa00-60c2-11eb-82b3-3c696ff5dbbb.png)

    Observe call chain latency changes.
![Screen Shot 2021-01-27 at 3 57 53 PM](https://user-images.githubusercontent.com/24226678/105968515-22e09e80-60c2-11eb-995b-7f3676dd3c0f.png)

    Observe alter information.
![Screen Shot 2021-01-27 at 4 01 52 PM](https://user-images.githubusercontent.com/24226678/105968532-27a55280-60c2-11eb-962c-c62d8f3c074a.png)

**Notes** need to provide a script or program to observe the above changes. The steps might be: 
1. Get dependencies.
2. Compare the dependency pod's metrics changes.
3. Check alert info. 

## API failure configuration file
### Example file
```
# cat api-failure.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: APIChaos
metadata:
  name: api-failure
  namespace: chaos-testing
spec:
  action: api-failure
  duration: '30m'
  responsecode: '504'
  responsebody: '{}'
  percentage: 100%
  external: 'www.xxx.com'
  port: 80
  endpoint: /external/test
```

In this file, the experiment will inject 504 error for 30 minutes, when calling `www.xxx.com:80/external/test` API, 50% API will return 504. Details of the parameters are the same as above.

### Steps to inject failure and verify the result
1. Apply api-failure.yaml
```
# kubectl apply -f api-failure.yaml
```
2. Open Skywalking and wait for 30 minutes, you will observe below changes in the dashboard.
The verify steps are the same as above.
