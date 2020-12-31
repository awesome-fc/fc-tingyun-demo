## Background
Under the general trend of cloud native, application containerization and serverlessization are being quickly accepted and selected by developers.
 
Alibaba Cloud FunctionCompute (FC), released in 2017, is one of the earliest Serverless Cloud Services in China. Function as a service (FaaS) frees developers from operations such as servers management, resource planning. 

FC supports sub-seconds rapid auto-scaling and organizations only pay for the actual usage. However, FaaS vendors abstract many concepts that DevOps are already familiar with, which poses several new observability challenges compared with Serverfull technologies:
* Lack of instance/container level metrics and insights, such as CPU, networking traffic
* Single responsibility functions are distributed with arbitrary concurrency that can become difficult to trace the individual requests
* Functions usually invoke other cloud services such as SQL/NoSQL databases or message queues.
* Cannot exec/ssh to an individual instance to perform performance profiling and other advanced investigation tooling.

Tingyun platform, one of the authoritative application performance monitoring platforms in China, is also always paying attention to the evolution of cloud native technology and new challenges in the field of observability. With observability capabilities of Tingyun APM , developers know well of the application by  instance-level monitoring, distributed tracing and link tracing.

In order to facilitate the integration of the third-party APM platform such as Tingyun, FC released [Lifecycle Capability](https://www.alibabacloud.com/help/zh/doc-detail/191143.htm) which allows APM agent to better monitor Serverless applications.

Next we introduce how to integrate Tingyun in FC, follow our steps and you will get your own serverless observability dashboard.

## Solutions to integrate with Tingyun
### Before you begin
1. Register Tingyun and get your licence key (You can get your licence key when you create application)
![tingyun](https://yqintl.alicdn.com/019bbd9d2b9d1dc697e630e77a6233ad2b938164.png)

2. Register FunctionCompute .

### Step 1: Prepare the environment
An easy and simple way to install Funcraft is to download the executable binary files.
1. Install Funcraft on the local machine. For more information, see [installation instructions](https://github.com/alibaba/funcraft/blob/master/docs/usage/installation-zh.md?spm=a2c63.p38356.879954.6.544d76e50rtte6&spm).
2. Run `fun --version` to check whether the installation is successful.
3. You need to configure funcraft with your own aliyun access key id and access key secret. Follow the steps in Configure Funcraft .Run `fun config` to configure Funcraft. Then configure Account ID, Access Key ID, Access Key Secret, and Default region name as prompted.
```
$ fun config
Aliyun Account ID 1234xxx
Aliyun Access Key ID xxxx
Aliyun Access Key Secret xxxx
Default region name cn-xxxx
The timeout in seconds for each SDK client invoking 300
The maximum number of retries for each SDK client 5
Allow to anonynously report usage statistics to improve the tool over time? (Y/n)
```

### Step 2: Deploy application
1. Run the following command to clone the sample project to your local machine.

    ```
    git clone https://github.com/awesome-fc/fc-tingyun-demo.git
    ```

2. Run the following command to access the cloned sample project:
    ```
    cd fc-tingyun-demo
    ```

3. Replace licence key in tingyun.json with your own license key
    ```
    {
      "nbs.app_name" : "my-ty-app",
      "nbs.agent_enabled" : true,
      "dataSentInterval": 1,
      "nbs.license_key" : "replace with your own key",
      "nbs.log_file_name" : "/tmp/tingyun.log",
      "nbs.audit" : true,
      "nbs.max_log_count": 3,
      "nbs.max_log_size": 9,
      "nbs.ssl" : true
    }
    ```
4. Run the following command to deploy the function:
    ```
    make deploy
    ```
5. The following result is returned:
    ```
    ➜  tingyun make deploy 
    docker build -t fc-go-runtime  -f build-image/Dockerfile build-image
    Sending build context to Docker daemon  2.048kB
    Step 1/5 : FROM golang:1.12.16-stretch
     ---> 7ad03a8aece5
    Step 2/5 : RUN mkdir -p $GOPATH/src/golang.org/x/
     ---> Using cache
     ---> 5678ac31beda
    Step 3/5 : RUN cd $GOPATH/src/golang.org/x/ && git clone https://github.com/golang/net.git
     ---> Using cache
     ---> 77d355f1730b
    Step 4/5 : RUN cd $GOPATH/src/golang.org/x/ &&  git clone https://github.com/golang/sys.git
     ---> Using cache
     ---> 414d3e786a8e
    Step 5/5 : RUN go get github.com/TingYunAPM/go
     ---> Using cache
     ---> 4c08b95694b5
    Successfully built 4c08b95694b5
    Successfully tagged fc-go-runtime:latest
    docker run --rm -it -v $(pwd):/tmp fc-go-runtime bash -c "go build -o /tmp/code//bootstrap /tmp/code/main.go"
    chmod +x code/bootstrap
    fun deploy -y
    using template: template.yml
    using region: cn-shenzhen
    using accountId: ***********3743
    using accessKeyId: ***********Ts6J
    using timeout: 20
    Collecting your services information, in order to caculate devlopment changes...
    Resources Changes(Beta version! Only FC resources changes will be displayed):
    ┌─────────────────────┬──────────────────────────────┬────────┬──────────┐
    │ Resource            │ ResourceType                 │ Action │ Property │
    ├─────────────────────┼──────────────────────────────┼────────┼──────────┤
    │ tingyun-integration │ Aliyun::Serverless::Function │ Modify │ CodeUri  │
    └─────────────────────┴──────────────────────────────┴────────┴──────────┘
    Waiting for service fc-integrate-with-third-apm to be deployed...
            make sure role 'aliyunfcgeneratedrole-cn-shenzhen-xxx' is exist
            role 'aliyunfcgeneratedrole-cn-shenzhen-xxx' is already exist
            Waiting for function tingyun-integration to be deployed...
                    Waiting for packaging function tingyun-integration code...
                    The function tingyun-integration has been packaged. A total of 2 files were compressed and the final size was 4.16 MB
            function tingyun-integration deploy success
    service xxx deploy success
    ```

Application has been deployed successfully, and your function can be monitored by tingyun. 

The application template has time trigger by dafault, it is triggerd every minite, and you can view a default dashboard with 1tps invocation.
You can also invoke function by `fun invoke` command or API calls.


### View dashboard 

![](https://img.alicdn.com/imgextra/i2/O1CN011fuha11izibKhhJvn_!!6000000004484-2-tps-1448-1212.png)
![](https://img.alicdn.com/imgextra/i4/O1CN01mWPElO1TDQbmeVtMr_!!6000000002348-2-tps-1414-1210.png)
![](https://img.alicdn.com/imgextra/i2/O1CN01vYgzuR1PnpcnJoy7a_!!6000000001886-2-tps-1430-1267.png)