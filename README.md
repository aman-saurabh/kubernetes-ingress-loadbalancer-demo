# kubernetes-ingress-loadbalancer-demo
A simple to deploy and demonstrate working of Ingress and load balancer with kubernetes

For help and reference you can visit following link :-

https://aws.amazon.com/blogs/containers/how-to-expose-multiple-applications-on-amazon-eks-using-a-single-application-load-balancer/

***************************************************************************************
For this project to work properly we need to ensure few things:

1.) For load balancer to work with AWS EKS cluster :- Install and setup AWS Load Balancer Controller for EKS cluster :- For this use the following link :-

https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html

But for it to work you will also need to Create an IAM OIDC provider for your cluster.You can use the following link for this purpose.

https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html

Apart from that we also need to set some tags in all subnets. You can check following links for help and details :

https://www.youtube.com/watch?v=3WbEt_sfTWU

https://aws.amazon.com/premiumsupport/knowledge-center/eks-load-balancer-controller-subnets/

https://aws.amazon.com/premiumsupport/knowledge-center/eks-vpc-subnet-discovery/

***************************************************************************************
Create a directory called green and another one called yellow.

mkdir green/ yellow/

Copy the code from each application and save it as index.html in their respective directories.

Green application:

echo '<html style="background-color: green;"></html>' > green/index.html

Yellow application:

echo '<html style="background-color: yellow;"></html>' > yellow/index.html

Let’s now create the Dockerfiles in the same directories as the index.html files.

Green application:

cat <<EOF > green/Dockerfile
FROM public.ecr.aws/nginx/nginx:1.20-alpine
RUN mkdir -p /usr/share/nginx/html/green
COPY ./index.html /usr/share/nginx/html/green/index.html 
EXPOSE 80
EOF

Yellow application:

cat <<EOF > yellow/Dockerfile
FROM nginx:alpine
RUN mkdir -p /usr/share/nginx/html/yellow
COPY ./index.html /usr/share/nginx/html/yellow/index.html 
EXPOSE 80
EOF

Let’s now create an Amazon ECR repository for each application and upload the respective Docker images.

aws ecr create-repository --repository-name green
aws ecr create-repository --repository-name yellow

Having created the repositories, access them in AWS management console in ECR section and select your repository and click on "View push commands" button.
It will open a page and will view you commands to push your docker image in the ECR repository.

We can use these commands as follows :
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ECR_REPO
------------------------------------------------------------
cd yellow/
docker build . -t yellow
docker tag yellow:latest $AWS_ECR_REPO/yellow:latest
docker push $AWS_ECR_REPO/yellow:latest
cd ..

cd green/
docker build . -t green
docker tag green:latest $AWS_ECR_REPO/green:latest
docker push $AWS_ECR_REPO/green:latest
cd .. 
------------------------------------------------------------
It will create docker image for yellow and green projects and will push the image in AWS ECR repository.  
Copy the URI of both images from AWS ECR repository.

Now define the environment configuration file in Kubernetes. We have defined in color-app.yaml file. So kubernetes related all data is defined in this file like Namespace, Deployment, Service, ingress etc.

Now Create the environment in AWS EKS cluster using following command.
kubectl apply -f color-app.yaml

Once ALB is provisioned, you can check the settings automatically made in ALB by going to the AWS Management Console in Amazon EC2 > Load Balancers > select the ALB > click the Listeners tab > click View/edit rules.

After the ALB is provisioned, run the command below and copy the DNS entry assigned to it. Paste the URL into your browser using /green or /yellow at the end, as shown in the images below:

kubectl get ingress color-app-ingress -n color-app -o=jsonpath="{'http://'}{.status.loadBalancer.ingress[].hostname}{'\n'}"

Let's suppose above command returns following output:
http://k8s-color-app-9d758f42ca9cdf2-377179600.ap-south-1.elb.amazonaws.com

So in browser enter following url to visit yellow page:
http://k8s-color-app-9d758f42ca9cdf2-377179600.ap-south-1.elb.amazonaws.com/yellow

And enter following url to visit yellow page:
http://k8s-color-app-9d758f42ca9cdf2-377179600.ap-south-1.elb.amazonaws.com/green









