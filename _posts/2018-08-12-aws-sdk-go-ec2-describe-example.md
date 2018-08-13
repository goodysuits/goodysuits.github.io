---
title: aws-sdk-go를 사용해서 ec2 instance 목록&정보 조회하기
key: 20180812
tags: go aws aws-sdk-go
---

<!--more-->

진행하던 go 프로젝트에서 aws ec2의 인스턴스를 조작해야 할 일이 생겨서, aws-sdk-go를 이용해 aws ec2 서비스의 API를 사용하기로 했다.  
이 포스트는 aws-sdk-go ec2의 기초적인 사용법과 인스턴스 목록을 가져오는 방법에 대한 설명이 포함되어 있다.

##### IAM user 설정
우선 ec2접속 권한이 있는 user를 생성하자.
기존 계정에 ec2 access 권한을 줘도 상관은 없지만 이 포스트에서는 새로운 user를 생성하기로 한다.

![screenshot1](/assets/images/2018-08-12/screenshot1.png)

![screenshot2](/assets/images/2018-08-12/screenshot2.png)

ec2 access 권한은 필요한 권한만 주는 것이 좋지만, 여기서는 FullAccess 권한을 주기로 했다.

![screenshot3](/assets/images/2018-08-12/screenshot3.png)

위와 같이 user를 생성하고 권한을 부여한 후, ACCESS KEY ID와 SECRET ACCESS KEY를 복사해놓도록 하자.

##### aws credentials 설정
ACCESS KEY ID와 SECRET ACCESS KEY를 사용해 credentials의 새로운 프로필을 설정하도록 하자.  
ACCESS KEY ID와 SECRET ACCESS KEY는 환경변수를 통해 aws-sdk-go로 넘겨줄 수도 있다.

~/.aws/credentials


````
[default]
aws_access_key_id=
aws_secret_access_key=

[ec2-user]
aws_access_key_id=AKIA
aws_secret_access_key=4dGp
````


해당 파일에 ec2-user라는 이름의 프로필을 생성했다.  
go app에서는 이 프로필을 사용해 ec2에 접근할 것이다.


##### go project에 적용


````go
package main

import (
    "fmt"

    "`github.com/aws/aws-sdk-go/aws`"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/aws/awserr"
    "github.com/aws/aws-sdk-go/service/ec2"
)

func main() {
    sess, _ := session.NewSessionWithOptions(session.Options{
        Config: aws.Config{
            Region: aws.String("ap-northeast-2"),
        },
        Profile: "ec2-user",
    })

    DescribeInstances(*sess)
}

func DescribeInstances(sess session.Session) {
    svc := ec2.New(&sess)

    name := "*test*"
    params := &ec2.DescribeInstancesInput{
        Filters: []*ec2.Filter{
            {
                Name:   aws.String("tag:Name"),
                Values: []*string{aws.String(name)},
            },
        },
    }

    result, err := svc.DescribeInstances(params)

    if err != nil {
        if aerr, ok := err.(awserr.Error); ok {
            switch aerr.Code() {
            default:
                fmt.Println(aerr.Error())
            }
        } else {
            fmt.Println(err.Error())
        }

        return
    }

    fmt.Println(result)
}
````


인스턴스의 name tag에 test라는 단어가 포함된 인스턴스를 모두 검색하는 api를 작성하였다.


````go
func main() {
    sess, _ := session.NewSessionWithOptions(session.Options{
        Config: aws.Config{
            Region: aws.String("ap-northeast-2"),
        },
        Profile: "ec2-user",
    })

    DescribeInstances(*sess)
}
````


session 패키지의 NewSessionWithOptions 함수를 사용하여 사용할 credentials의 프로필 이름과, region을 지정한다.

위에서 언급한 것 처럼 access key id와 secret access key를 환경변수로 넘겨주는 방식을 사용하고 싶다면, 아래와 같은 방식으로 사용할 수 있다


````go
func main() {
    os.Setenv("AWS_ACCESS_KEY_ID", "AKIA")
    os.Setenv("AWS_SECRET_ACCESS_KEY", "4dGp")
    sess, _ := session.NewSessionWithOptions(session.Options{
        Config: aws.Config{
            Region: aws.String("ap-northeast-2"),
        },
    })
}

// os 패키지를 import해주어야 한다.
````


````go
func DescribeInstances(sess session.Session) {
    svc := ec2.New(&sess)

    name := "*test*"
    params := &ec2.DescribeInstancesInput{
        Filters: []*ec2.Filter{
            {
                Name:   aws.String("tag:Name"),
                Values: []*string{aws.String(name)},
            },
        },
    }

    result, err := svc.DescribeInstances(params)

    if err != nil {
        fmt.Println("DescribeInstances Fail")
    } else {
        fmt.Println(result)
    }
}
````


생성된 session을 이용하여 ec2 client를 생성한다.
그리고 인스턴스 목록을 가져오기 위해 DescribeInstances를 사용하는데, 그 인자로 넘겨주기 위한 설정들을 DescribeInstancesInput struct를 이용해 생성한다.

위의 예제는 name tag중 test라는 단어가 포함된 모든 인스턴스의 목록을 가져오도록 작성되어 있다.
name tag가 다른 정보를 통해 인스턴스 목록과 정보를 조회하려면 아래 문서를 참조하자.  
[aws-sdk-go ec2 DescribeInstancesInput](https://docs.aws.amazon.com/sdk-for-go/api/service/ec2/#DescribeInstancesInput)  
만약 instance의 id를 통해 instance의 정보를 가져오고 싶으면 Filters가 아닌 InstanceIds를 이용하면 된다.


````go
params := &ec2.DescribeInstancesInput{
    InstanceIds: []*string{
        aws.String("i-1234567890abcdef0"),
    },
}
// aws.String을 사용하려면 github.com/aws/aws-sdk-go/aws 패키지를 import해주어야 한다.
````


모든 instance의 목록을 가져오려면 DescribeInstances의 인자로 nil을 넘겨주면 된다.


````go
result, err := svc.DescribeInstances(nil)
````


go app을 실행하면 아래와 같은 결과의 구조는 아래와 같다.


````
{
    Reservations: [{
        Instances: [{
            ...
        }, {
            ...
        }],
        OwnerId: "12345678901234",
        ReservationId: "r-1234567890abcdefg"
    }]
}
````


결과는 Reservations의 배열로 이루어져 있고, 각 Reservation마다 Instances의 배열들이 들어있다.  
현재 Reservations의 뜻은 정확히 알지 못하므로, 추후 알게 된다면 설명하도록 하겠다.

result.Reservations[i].Instances[j]와 같이 접근하여 해당 instance의 정보를 이용할 수 있다.
