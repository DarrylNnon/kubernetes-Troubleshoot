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


## option3: Unbound Persistent Volume

It means my pod is on the pending status, for some reason my pod is not getting assign to a machine.

![image](https://github.com/user-attachments/assets/89981626-8b30-48ac-ace6-afa65712fbb8)

again like we say in order to troubleshoot i have to use these two commands:

```sh
kubectl get pods --watch
kubectl describe
```
![image](https://github.com/user-attachments/assets/b266b671-f5ac-4b92-8e59-e7624b72a900)

Now i look at the event, it clearly say my pod is not getting assign to a machine because it didn't find any persistent volume 

![image](https://github.com/user-attachments/assets/df8c5bc1-a9b0-4d9c-af4d-c32591a0753f)

As i learn in a classe, as soon that i assign a pod, what will happen is that if my pod has a persistent volume to be attach to my container depending on the application then while the pod is getting assign it will also make sure that the persistent volume is attach to the pod, only then my pod will go to even pod download. This is because when am trying to create a container i need to make sure whatever is necessary  for the application to run it should be available.my persistent volume is needed to copy some file from logs in that case before my container start, it should have a persistent volume.

![image](https://github.com/user-attachments/assets/bcae7ab3-a083-4a0d-be1b-d799b7b9670c)

Whe i run this file, it look for all the requirement that i specify in my yaml file.

To quickly check, i can run this command :

```sh
kubectl get pvc
```
![image](https://github.com/user-attachments/assets/d5173351-bd7c-4e18-8a48-2e6b57c117d4)

i can see the claim is on pending status cause i dont have a persistent volume attach.Then in order to fix it, either am the administrator or a dev i have to manually make sure i have persistent volume claim attach to my pod and if there is a searate team whorking on it, i should tell them, hei guys you need to attach a persistent volume in your pod of 1Gi

There could be another reason some time, when i try to build a dynamic pv for that there's another object call "storage class"

```sh
kubectl get sc
```

![image](https://github.com/user-attachments/assets/264bdc99-e229-4e52-a438-a8e55a45bd89)

so this storage will do per examaple if i create a claim tehn automatically it will go to aws and create an aws storage and that ebs storage will be map to my cluster. So that's how dynamically i can create it.

At the moment, you can see i haven't set any storage that's why it's using the default storage which is not able to create any persistent volume. So the solution for this is to either make sure there is already a pv available if not i need to set my storage class (sc) in such a way that  it can dynamically connect to a cloud like AWS and create a storage and a persistent volume togheter.That's how am going to resolve this issue.

![image](https://github.com/user-attachments/assets/151bd25f-ffd8-4857-b9ad-e6372737e9c9)

## option4: Node Taint

```sh
kubectl get pods --watch
kubectl describe
```
![image](https://github.com/user-attachments/assets/4c53a993-49cf-4a8f-84a1-ad0678e83031)

As i can see the pods is also on pending state which mean my pod for some reason has not been assign to a machine. And here it's call a "node taint"

Let's use "kubectl describe pod"

![image](https://github.com/user-attachments/assets/756f84ad-4ec9-487e-817f-310771d46705)

![image](https://github.com/user-attachments/assets/41989003-dade-4639-8d56-450911ebceaa)

As i can see, the event is saying that in my cluster there's only one machine and that machine has a property call "taint".

So i go back into my advanced kubernetes class, and there is a concept call "taint" so taint is just a label that i can apply on a machine. So if i apply a label call taint in any machine what will happen is that i can't create a pod on it. So the only way i can create a pod on a mahcine which has taint is i need to use a pod property call "toleration" or "tolerate" which is matching the same label.

![image](https://github.com/user-attachments/assets/0808114c-8e00-476f-a25f-dc4b44dd7b52)

So lets' go to my example 

![image](https://github.com/user-attachments/assets/0cc8b214-455c-4a05-a0d2-51bf08b3d282)

and as you can see i don't have any label apply so there's only one machine am trying to create and this is where is i comeback and look at my issue:

![image](https://github.com/user-attachments/assets/9836974a-801f-4868-9610-b9a2023d54f9)

How do i find that a machine has a taint or not i can do:

```sh
kubectl get nodes
kubectl describe node
```

![image](https://github.com/user-attachments/assets/67311159-277f-4364-9cd2-9184e26d52ef)

So this is where i need to describe a node not just a pod 

![image](https://github.com/user-attachments/assets/56e75ba0-1241-4b0f-b904-951fb02099f4)

As i can see my machine that i have in the cluster has already apply with a label call "taint=true". So now if i want pod to get to this machine only then in my "pod spec" or under the container i need to add a property call "toleration" and that should exactly match the same label of the taint call "taint=true".

So this is how only if i have the toleration taint in my pod matching the same pod of the machine, then my pod will be allow to be create on a machine. If not taint will make sure no pod is getting assign to that machine. So this is another scenario where my pod is on pending phase.

![image](https://github.com/user-attachments/assets/d81db6b2-0a58-4504-a427-4137a6061869)

Now let see scenario what happen when the pod getting aasign to a machine.
so that's where image can be download and start running a container.

## option5: Unavailable Configmap

As usual i use the command below:

```sh
kubectl get pods --watch
kubectl describe
```

![image](https://github.com/user-attachments/assets/f2d48e1e-3d0f-48b6-b9b1-44988414c6ed)

Now am moving to the next phase, once the pod is getting assign to a machine, if there's an issue the container creation then what are the differents scenario i will get.

Good, so now i can see that pod is on the pending status but is moving to the containercreation. so this clearly indicate that my pod get schedule or assign to a machine. But after that it's trying to create a container, there's some steps that show that something is wrong. So how do i find out what is going on?

![image](https://github.com/user-attachments/assets/ed6ea93e-a036-45d0-8e7c-7f8859cb1152)

Once again i use the "kubectl describe"

![image](https://github.com/user-attachments/assets/5c275ca6-fb39-43e1-b7ec-b52c0fcb0341)

![image](https://github.com/user-attachments/assets/fccfd023-5521-496b-ac67-855cc6bc31d9)

Then if i look down on the event, it clearly say first my pod get assign to a machine but after that there's something call configmap which is not found.

So again let go back to my advanced kubernetes class.

Suppose if i want property file or application file to be injected in my container then i define the property while am creating the pod to take the file from a configmap object and injecting into the pod.

So once again, before the container is getting created, my pod should have the reference of configmap file because only then it can mounted as a volume inside my container.

So i assume my pod is created without this configmap file available then the application will any how failed. So tha't where if i had specify to use configmap and to mounted inside my container on a particular folder and if that is not  available then my container will not even be created.

Let's see my example:

![image](https://github.com/user-attachments/assets/6f1b6a0d-7c28-4ca4-b1cc-4ec29492255b)

As i you can see, am telling while my container is created i want a folder call "/tmp/config" in which i want a file call "sample.conf" coming from an object call "configmap" so this is how automatically when am creating a container my config.file will be available and my application will start. so that's why we're havingthis issue

![image](https://github.com/user-attachments/assets/51d485c3-3b37-4f83-8b47-3783110497a1)

and to check once again i do: "kubectl get pods --watch" then:
```sh
kubectl get cm (configmap)
```
![image](https://github.com/user-attachments/assets/0f26bf0b-347d-4c52-ad62-f4ffa5b7ff2b)

As you can see, now there's a default configmap but the configmap that i need is call "myMap" and that's not available...that's why my pod is in creating status.

So to dibugg i make sure i have the configmap created and in the configmap i have correct file associated only then while the container is creating this will be injected. So remember only when all these steps are done, only then the container will be created.

![image](https://github.com/user-attachments/assets/e9cfc5d2-08ae-4011-a1b7-8f0eb7f6205a)

Now we've seen 5 options, let see another option

## option6: Unavailable Secret
