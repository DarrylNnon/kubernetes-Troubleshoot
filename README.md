# kubernetes-Troubleshoot
Hands-on practical techniques of how to troubleshoot kubernetes easily  13 Real-world scenarios 

![image](https://github.com/user-attachments/assets/45b62886-90fd-446d-abd7-a84dbff226d5)

Kubernetes is a crucial concept in devops,cloud and it's important to know how to troubleshooting it.

# POD STATUS
Before jumping into to debugging, i have to understand what is the state of my pods?
out of these three, where is the state of my pods? then i can proceed with the troubleshooting

![image](https://github.com/user-attachments/assets/e7d116b6-5557-4c17-8cce-0332ed1c781f)

I will work on 13 real-world scenario to troubleshoot my kubernetes issue

## option1: Insufficient Resource

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

## option2: Node Affinity

My pod is not getting assign to a machine and it's on the status "pending"

![image](https://github.com/user-attachments/assets/b65b7816-c0a9-4736-b754-09846d80fe9d)

What could be the reason?
This scenario is call node affinity.
Then i will do once again 
```sh
kubectl get pods --watch
kubectl describe
```
to see the issue

![image](https://github.com/user-attachments/assets/e30de342-aeb3-49ce-b142-e6d9bd05a756)

on the events is It's telling me that the scheduler didn't find any machine in my cluster and because of that it's not able to assign or schedule the pod

![image](https://github.com/user-attachments/assets/af99989e-8f5c-4ed6-94c3-0b8ea5665b42)

and the reason are pod are the node affinity or the selector

- Let's open our node-affinity.yml file

![image](https://github.com/user-attachments/assets/08a5c690-ed8a-4a87-8a0d-99d11bff8ea3)

The solution is to find out what is the right machine to create this pod and on that machine , i have to create "Label" for which i can run certains commands like kubectl label node and the label name and what label i want.

First i will do is to:
```sh
kubectl get nodes --show-labels
```
![image](https://github.com/user-attachments/assets/f3ce632e-25db-4ccc-901f-e0d9a8aac493)

out of all this labels i do not see the label i want which node..so i have to go ahead and apply the label and i will see the label getting assign 
```sh
kubectl get pods --watch
```
![image](https://github.com/user-attachments/assets/616cacfe-f194-4b48-abb1-3bbc6ffe1069)

I will take the same script to avoid doing it manually, i will go ahead an apply command write "y" and boom this is what happen

![image](https://github.com/user-attachments/assets/24c71576-1616-4014-aa9c-190cba4537e7)

As soon that i apply the label on that particular machine, it will move from pending to to container creation and container running now.

![image](https://github.com/user-attachments/assets/203fe15d-c02c-40ff-ab1f-9482b2702dab)

So now when i do again 
```sh
kubectl get pods --watch
```
I can now see the running container pod

![image](https://github.com/user-attachments/assets/a058790d-83d3-41a0-9669-fe8af16af1a5)

```sh
kubectl describe
```
![image](https://github.com/user-attachments/assets/0fe8d01e-6c59-4105-963c-60b9c2f70792)

As soon i apply the label, then kubernetes find that there is machine matching the label and that is the node affinity and it then assign the pod. So this where once the pod get assign and the image will be downloaded and then on that machine it  will create go to start the container. and only when it comes, i can now see the status

![image](https://github.com/user-attachments/assets/90002765-044c-4496-806b-81acd25cd740)


![image](https://github.com/user-attachments/assets/16af9eb9-f9be-45f6-8fc6-2615b116b47d)

Now we have to delete it by wrting "y" after practicing

![image](https://github.com/user-attachments/assets/9da52691-3a27-4b00-b99f-ffdd977bf58e)

The pod will also got deleted

![image](https://github.com/user-attachments/assets/96e0a96d-5ade-49f0-9d53-ec3bc146fd51)

![image](https://github.com/user-attachments/assets/7135c66c-d966-4168-adbc-3e63ddbe7472)

