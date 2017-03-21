clear all;close all;clc;
cd('C:\Users\hasee\Desktop\图像\错台线2\pic_data');
files=dir('*.bmp');
m = size(files,1);
P=zeros(m,1);%建立m*1的空矩阵,存入一个角度特征
T=zeros(m,4);
Y0=[0 0 0 0];%Y0-Y3四种目标矩阵
Y1=[0 0 0 1];
Y2=[0 0 1 0];
Y3=[0 0 1 1];
%%
for i=0:39
    for j=0:3
    n = (i*4+j);
    T(n+1,:)=eval(strcat('Y',num2str(j)));%读入矩阵并存入H中
    %img=double(imread(strcat(num2str(j),'_',num2str(i),'.bmp')));%读入图像
    str = strcat(num2str(j),'_',num2str(i),'.bmp');
    img=double(imread(str));
    [L,k] = bwlabel(img,8);%n白色区域数
    boxes = imOrientedBox(L);
    imshow(img),title('角度数值（单位-度）') 
    hold on
    for ii = 1:k 
        text(boxes(ii,1)+10,boxes(ii,2)+10,num2str(180-boxes(ii,5)*180/pi),'color','b');
        num=double(180-boxes(ii,5)*180/pi);
        num=num/360;
        P(n+1,:) = P(n+1,:)+num;%将特征值存入矩阵R中
    end
    hold off 
    end
    close all
end
size(P)
size(T)
net_1=newff(P',T',10,{'tansig','purelin'},'traingdm');

%  当前输入层权值和阈值 
%inputWeights=net_1.IW{1,1};
%inputbias=net_1.b{1};
%  当前网络层权值和阈值 
%layerWeights=net_1.LW{2,1};
%layerbias=net_1.b{2};

%  设置训练参数
net_1.trainParam.show = 50; 
net_1.trainParam.lr = 0.05; 
net_1.trainParam.mc = 0.9; 
net_1.trainParam.epochs = 10000; 
net_1.trainParam.goal = 1e-3; 

%  调用 TRAINGDM 算法训练 BP 网络
[net_1,tr]=train(net_1,P',T'); 

%  对 BP 网络进行仿真
A = sim(net_1,P'); 
%  计算仿真误差  
E = T' - A; 
MSE=mse(E) 

[file,path] = uigetfile('*.bmp;*.jpg;*.png','选择一幅图片');
img= double(imread(fullfile(path,file)));
[L,k] = bwlabel(img,8);%n白色区域数
boxes = imOrientedBox(L);
imshow(img),title('角度数值（单位-度）') 
hold on
for ii = 1:k 
    text(boxes(ii,1)+10,boxes(ii,2)+10,num2str(180-boxes(ii,5)*180/pi),'color','b');
    num_1=double(180-boxes(ii,5)*180/pi);
    num_1=num_1/360
    end
Reslut=sim(net_1,num_1);
for jj=1:4
    if Reslut(jj,:)<0.5
        Reslut(jj,:)=0;
    else
        Reslut(jj,:)=1;
    end
end
Reslut
