# kubernetes-Troubleshoot
Hands-on practical techniques of how to troubleshoot kubernetes easily  13 Real-world scenarios 

![image](https://github.com/user-attachments/assets/45b62886-90fd-446d-abd7-a84dbff226d5)

Kubernetes is a crucial concept in devops,cloud and it's important to know how to troubleshooting it.

# POD STATUS
Before jumping into to debugging, i have to understand what is the state of my pods?
out of these three, where is the state of my pods? then i can proceed with the troubleshooting

![image](https://github.com/user-attachments/assets/e7d116b6-5557-4c17-8cce-0332ed1c781f)

I will work on 13 real-world scenario to troubleshoot my kubernetes issue

## --option1 Insufficient Resource"; manifest="insufficient-resources.yml

First how do i know the status of the pod that am trying to find? I have to run
```sh
kubectl get pods --watch
./runme.sh -option1
```
![image](https://github.com/user-attachments/assets/135c5148-0224-4c64-b37b-5604f9cde25b)

After looking at the pods i notice the status of the pod is pending, it mean that my pods for some reason could not be assign to a machine. and to get the exact problem, now i have to run another command: 

```sh
kubectl describe
```
![image](https://github.com/user-attachments/assets/97a18315-f8fe-47f7-860a-bec6d20423e8)

then scroll down and look for events that will me understand what's happening:

![image](https://github.com/user-attachments/assets/a0890296-f6d0-4c86-9dbb-9539943ad268)
Here it say my pod need some memory but that memory is not available in any of the cluster
- Let's open our yml file insufficient-resource.yml to see the content
  
![image](https://github.com/user-attachments/assets/9404a6a1-2dea-4317-9158-057e23ea2b69)

When i define my pod i have to define the minimum capacity and the limits...and limits is the maximum resource it can run on a particular machine.then when i apply the "kubectl apply -f insuficient-resource.yml" it will look for my memory and limit to see if i have the require capacity specify in order to proceed.

In my scenario my machine has less than 4Gi that's why it's giving me insufficient memory.
In order to fix i have to look the machine i have, also i can use auto scale which will automatically scale up or down everytime i have this issue.( this will only work on a cloud environment)

In my local environment, i have to do it manually.

Also talk with the dev and ask if we truly need 4Gi or what is the minimum capacity that's need for this pod to run.and find out if i have to have more worker node

![image](https://github.com/user-attachments/assets/f0fc313a-9e1a-4448-addb-b26d4fa83496)

## Option 2: 
