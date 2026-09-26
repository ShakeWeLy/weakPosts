---
title: 2026年7月28日 MoME learn
date: 2026-07-28 22:30:44
tags: 

---



# `MoME`汇报



# nnUnet`汇报

## 0 研究背景和痛点

> ***\*As different types of\**** ***\*lesions exhibit distinct characteristics on different imaging\**** ***\*modalities, segmentation methods are typically developed\**** ***\*in a task-specific manner, where each segmentation model\**** ***\*is tailored to a specific lesion type and modality.\****
>
> ***\*use of task-specific models requires predetermination\**** ***\*of the  lesion type  and  imaging  modality, which  compli\*******\*cates their deployment in real-world scenarios.\****
>
> ***\*a universal foundation model for brain lesion\**** ***\*segmentation on\**** ***\**magnetic resonance imaging\**\*** ***\*(MRI),which\**** ***\*can automatically segment different types of brain lesions\**** ***\*given input of various MRI modalities. ****
>
>  ***\*a novel\**** ***\**Mixture of Modality Experts\**\*** ***\*(MoME)framework with multiple\**** ***\*expert networks attending to different imaging modalities.\**** 

> Since brain lesions tend to manifest with different characteristics due to various patient-dependent pathological alterations, the segmentation of different types of brain lesions requires the use of different **magnetic resonance imaging** (MRI) modalities
>
> used standalone or as a combination [12],[13],  [14],  which  increases the  complexity  of segmentation model development. 
>
>  for  different  types  of  brain lesions,  different task-specific models need to be developed. Fig. 1a) compelled to manually choose 

> 1 propose a novel  task-agnostic  universal  foundational  model  (Fig.  1b), which is capable of automatically segmenting various types of brain lesions regardless of the input MRI modality. The model can also be extended to handle heterogeneous inputs with multiple modalities.
>
> 2 Foundation models, due to the complexity of medical
>  imaging data and tasks, the segmentation accuracy of such a
>  universal model is often limited focusing on narrower scopes are explored, which are designed for  a  particular  modality  or  a  particular  organ/task   most existing foundation  models  may  not  be  directly  applicable  to  brain lesion  segmentation  requires the capability of han dling the diversity of input imaging modalities, which is not
>  addressed by many of the existing foundation models.

> Unlike anatomical structure segmentation, brain lesion segmentation deals with a broader variations of data distributions, in terms of different lesion types, varying spatial locations, scattering of multiple lesions, and irregular shapes [1], [32], which poses substantial challenges for developing segmentation foundation models.


## 1 核心方法

> 1 ***\*A hierarchical gating network is proposed to combine the\**** ***\*expert predictions and foster expertise collaboration.\**** 
>
> ***\2 *to avoid the degeneration of each expert  network,\**** ***\*we introduce a curriculum learning strategy during training\**** ***\*to preserve the specialisation of each expert.\**** 
>
> 3 ***\*to handle the combination of multiple input modalities,we propose MoME+,which uses a soft dispatch network\**** ***\*for  input  modality  routing.\****  

> MoME is developed to be task-agnostic and capable of handling diverse imaging modalities and accommodating various types of brain lesions. Each expert specialises in processing certain image modalities.
>
> • To explore collaboration among the experts, a hierarchical gating network is designed to combine expert predictions and facilitate collaborative expertise exploration layer by layer. Additionally, to keep the  specialisability of each expert, an anti-degeneration strategy based on curriculum learning is developed.
>
> • We conducted comprehensive experiments on nine brain imaging  datasets  comprising  a  total  number  of  6,585 annotated 3D images, with five MRI modalities and eight lesion types. The results show that MoME outperforms competing  foundation  models  in  multiple  aspects  and provides improved generalisation onto unseen datasets.

>  The MoME model has been extended to MoME+ to handle combined multiple modalities as input. \
>
> develop a novel dispatch network with a soft assignment strategy and prior constraints, and employ random dropout of modalities during  training  to  mimic  the  heterogeneous  input.  
>
> include new experimental results for handling missing modalities, investigate strategies of  decoder  sharing  for  multiple  experts  to  improve  model efficiency, as well as extend the illustration and discussion.

>  1 the definition of experts in  the  MoE  model  could  be  carefully  tailored  to  specific tasks and data domains. Brain lesion  segmentation involves multiple imaging modalities. Several studies have shown that utilising the modality information benefits the  segmentation performance  [46],  [47].  There  is  potential  to  leverage  the characteristics of each modality to define the expert for each modality under the MoE framework.
>
> 2   modality  missing  problem,  Consequently, numerous input-specific segmentation models need to be trained, dedicated to each modality combination [14], [53]. Wang et al. proposed non-dedicated training, which results in **N** models and reduces the model complexity [12]. The non-dedicated training approach, while requiring fewer models for training, may compromise performance to some extent compared to dedicated training 





### 1.1 MoME 

是什么? 是模态个数N的N个专家网络

为什么? 每一个专家被设计为专门处理某一模态的任务

怎么做? 3D网络, 图片输入给所有专家, 之后会在进行G网络处理(each expert network and the gating network using the nnU-Net architecture)

> 1 The single input image will go through all expert networks, with their outputs ensembled by the gating network to form the final output. 
>
> 

### 1.2  **Hierarchical Gating Network**

是什么? 是对每一个专家的在某一个分辨率下的输出进行加权求和

为什么? 因为其他模态的对目标模态具有互补(provide  complementary  information  that  both  contribute  to lesion segmentation.)

怎么做?  加权求和. 推理阶段只输出最高分辨率的(each expert network and the gating network using the nnU-Net architecture)

> 1 Following the specialisation of  expert  networks,  we  propose  a  gating  network  **G** that facilitates the collaboration and conducts voxel-wise weighted ensemble  among  the  expert  outputs.  Note  that  different imaging modalities can be correlated while providing complementary information. 



### 1.2  **Alleviate Expert Degeneration

是什么? 训练过程中, 专家网络可能出现退化现象, 具体来说就是(where each expert no longer focuses on a specific modality and their activation becomes imbalanced, reducing the capability of the whole model to handle diverse modalities)

为什么? 

怎么做?    a  curriculum  learning , 训练过程中从整个网络, 逐步走向个体优化. 

具体来说?  a dynamic loss function: **f** epoch  is a dynamic weight that evolves with the  epoch  iteration.  **f** epoch   is  designed  as  **f** epoch    = **(**1 − epochcurrent **/**epoch total**)**2 , 

resolution level loss: the combination of the soft Dice loss and cross-entropy loss 并对每一个分辨率进行加权(kl)求和

专家loss: 同样原理, 不是直接e=(专家ouput), 只是变成了o=(e乘上g=(Gating network))





### 1.4  **Extended for Multiple **

> To  address  the  mismatch  between the  number  of  available  input  images  and  the  number  of expert networks, and to leverage information from available modalities to infer channels for missing modalities, we develop an additional dispatch network **D** , which generates a **N** × **N** dispatch matrix, and then dispatches the input images to the**N** expert networks.

是什么?  解决部分模态丢失

为什么?  模态信息输入可能少于设定的N

怎么做? a   **N** × **N**  trainable  network D.  (In this way, the dispatch network **D** learns to leverage the available modalities and perform a weighted combination of them to compensate for the missing modalities [55]. )( utilises three layers of fully connected  layers  to  achieve  its  lightweight  task, the numbers of channels are 4,  8, and  16  sequentially, )

1.输入全部模态的iamge Xi; 如果丢失则为zero

2.D为受限制的网络: ( D⋆i,j in the matrix D⋆ denotes the weight of the input image Xj contributing to the output image S i  ): 当有模态输入的时候, 输入自身输入, 没有模态数据的时候, 会进行而外的Dij(learned)处理输出混合数据

3.loss函数: (Additionally, the modality available information ***\**c\**\*** is treated as prior information embedded into the bottleneck of the gating network. Specifically, before embedding, we expand ***\**c\**\*** into a one-hot tensor of shape R**N**×**d**3, where **d**3  matches the shape of the 3D feature map at the bottleneck layer of the gating network.)

4.训练策略: 随机丢弃模态.



## 2 实验设计与结果

> ***\*evaluated  the  proposed\**** ***\*method on nine brain lesion datasets, encompassing five\**** ***\*imaging modalities and eight lesion types.The results show\**** ***\*that our model outperforms state-of-the-art universal mod\*******\*els for brain lesion segmentation and achieves promising\**** ***\*generalisation performance onto unseen datasets.\****

>demonstrate promising segmentation performance that is agnostic against modalities  and  modality  combinations.  Since  only  a  single universal model is needed, it eases model management and deployment,  making  it  applicable  to  a  broader  range  of scenarios. 

### 2.1 **Dataset and Task Description**

> We  curated  six publicly  available brain  lesion  segmentation  datasets  (MSSEG  [60],  OASIS  [61],  ISLES2022  [62], BraTS2021   [63],  WMH2017   [33],   and  ATLAS2 .0   [64]) and   three   in-house   datasets   (Tumour1,1    Tumour21 ,   and WMHsMix2 ), amounting to a total number of 6,585 3D images with brain lesion annotations. The dataset detail is reported in Table I. All images  were rigidly registered to the MNI152 template using ANTs [65]. Brain masks were applied to the images for skull removal. 
>
> For assessing the extended multi-input MoME+ model, we used the BraTS2021 dataset, which contains T1w, T1ce,  T2w,  and  FLAIR  modalities,  and  simulated  different scenarios  of  missing  modalities.
>
> For  heterogeneous  multi-modality  input,  the  competing foundational  models  cannot  be  applied.  We  compared  theMoME+ model with 2**N** − 1 dedicated nnU-Nets, each trained for one combination of modalities. 

### 2.2  **Competing Task-Specific and Foundation Models**



### 2.3   **Performance on Seen Datasets**

>  compares the  average Dice  score  and Hausdorff distance  (HD)  (mm) reports the overall GPU memory usage and GPU hours for training each model. 
>
>  report the Dice score with different levels of averaging: across the  14  seen tasks (task-level), across 6 seen datasets (dataset-level), and across 1,230 test images (image-level).
>
> achieving 2% to4% higher Dice  score than nnU-Net, Multi-Talent, Hermes, and nnU-Net ResEnc XL.It outperforms SAM-Med3D by a large margin of 7% to  15%. T
>
> The performance of MoME is comparable to task-specific nnU-Nets. However, the latter requires the training of 14 separate models, and its total amount of GPU memory consumption is about three times that of MoME.
>
> seven out of the eight. the Dice metric is lowest for T1w images, likely
>  because detecting lesions is more challenging on T1w, where
>  lesions do not manifest as clearly as on FLAIR and T2w.
>
> It shows that MoME achieves a more
>  discriminative clustering for images of different modalities and
>  lesion types compared to the single nnU-Net, which may be
>  relevant to its improved performance.

### 2.4   **Generalisation to Unseen Datasets**

> On  two  unseen  datasets  (Tumour1  and Tumour2), it even exceeds the task-specific nnU-Nets, which can be considered the upper-bound benchmark  as they had access to the training data from the unseen datasets. On the other unseen dataset (WMHsMix), MoME still performs relatively well and achieves an average Dice  score of 0 .7015, where  the  dataset  contains  diverse  WMH  lesions  of  five diseases

### 2.5 **Summary of MoME Results** 



### 2.6 消融实验

>  For the modality experts,✘***\*/\****✔ denotes randomly initialised experts versus  specialised experts based on the imaging modality; for the hierarchical gating,  ✘***\*/\****✔ denotes  the  basic  gating  without  hierarchical structure versus our hierarchical gating; and for the curriculum learning, ✘***\*/\****✔ denotes the application of Eq. (4) versus Eq. (2) during model training. When the three contributions are sequentially removed, even though the number of model parameters remains constant in this ablation  study, the  segmentation performance tends to decrease gradually, confirming the  benefits  of  these  contributions.  This  indicates  that  the improvement in MoME is not solely due to the larger model size, but also due to the modality-based expert design and the training  strategy, which better balances expert collaboration and specialisation within the MoME framework.

> We investigated the issue of expert degeneration by comparing the expert output, weighted by gating, without and with the proposed curriculum learning strategy. 
>
> without an anti-degeneration strategy, T2w and FLAIR experts lose their acquired modality knowledge,with the anti-degeneration  strategy, expert degeneration is effectively avoided. 
>
> each expert is activated according to its specialised modality or related modalities
>
> some modality experts exhibit a strong relationship and can be activated in tandem, such  as  T1w  and  T1ce,  

## 3 讨论与启发

> 
