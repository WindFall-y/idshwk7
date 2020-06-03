用matlab实现：

嵌入隐藏文本：
function [ste_cover,len_total]=rand_lsb_hide(input,file,output,key)
%读入图像矩阵
cover=imread(input);
ste_cover=cover;
ste_cover=double(ste_cover); 
%将文本文件转换为二进制
f_id=fopen(file,'r');
[msg,len_total]=fread(f_id,'ubit1');
%判断嵌入的信息量是否过大
[m,n]=size(ste_cover);
if len_total>m*n
  error('嵌入信息量过大，请重新选择图像');
end
%p作为消息嵌入位计数器
p=1;
%调用随机间隔函数选取像素点
[row,col]=randinterval(ste_cover,len_total,key);
%在LSB隐藏信息
for i=1:len_total
    ste_cover(row(i),col(i))=ste_cover(row(i),col(i))-mod(ste_cover(row(i),col(i)),2)+msg(p,1);
    if p==len_total
        break;
    end
    p=p+1;
end
ste_cover=uint8(ste_cover);
imwrite(ste_cover,output);
%显示实验结果
subplot(1,2,1);imshow(cover);title('原始图像');
subplot(1,2,2);imshow(output);title('隐藏信息的图像');

随机数生成算法：
function [row,col]=randinterval(matrix,count,key)
%计算间隔的位数
[m,n]=size(matrix);
interval1=floor(m*n/count)+1;
interval2=interval1-2;
if interval2==0
    error('载体太小，不能将私密信息隐藏进去');
end
%生成随机序列
rand('seed',key);
a=rand(1,count);
%初始化
row=zeros([1 count]);
col=zeros([1 count]);
%计算row,col
r=1;
c=1;
row(1,1)=r;
col(1,1)=c;
for i=2:count
    if a(i)>=0.5
        c=c+interval1;
    else
        c=c+interval2;
    end
    if c>n
        r=r+1;
        if r>m
            error('载体太小，不能私密信息隐藏进去');
        end
        c=mod(c,n);
        if c==0
            c=1;
        end
    end
    row(1,i)=r;
    col(1,i)=c;
end


提取隐藏文本信息：
function result = rand_lsb_get(output,len_total,goalfile,key)
ste_cover=imread(output);
ste_cover=double(ste_cover);
%判断嵌入信息量是否过大
[m,n]=size(ste_cover);
if len_total>m*n
  error('嵌入信息量过大，请重新选择图像');
end
frr=fopen(goalfile,'a');
%p作为消息嵌入位计数器,将消息序列写回文本文件
p=1;
%调用随机间隔函数选取像素点
[row,col]=randinterval(ste_cover,len_total,key);
for i=1 :len_total
    if bitand(ste_cover(row(i),col(i)),1)==1
        fwrite(frr,1,'ubit1');
        result(p,1)=1;
    else
        fwrite(frr,0,'ubit1');
        result(p,1)=0;
    end
    if p ==len_total
        break;
    end
    p=p+1;
end
fclose(frr);


命令行输入result = rand_lsb_get('test.bmp',136,'recover.txt',2020);
隐藏的文本信息即可被提取到文档recover.txt中
