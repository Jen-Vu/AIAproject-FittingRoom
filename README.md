# AIAproject-FittingRoom
<div align="center">
 <img src="image/result.png" width="700px" />
 <p>Fig.1 Testing result for different types of Try-On task.</p>
</div>

- Virtual Try-On scheme based on cycleGAN.
- Top3 project in Taiwain AI Academy finals.(Technical Professionals Program)
- Utilized cycle consistency to improve unrealistic results caused by either geometric matching or behavior of networks without inverse mapping.
- Code was developed and tested with pytorch==0.4.1, torchvision==0.2.1.
- _TOC_
   - [Dataset](#Dataset)
   - [Discussion](#Discussion)
     - [Geometric Matching Limitation](#Geometric-Matching-Limitation)
     - [Arms Missing Problem](#Arms-Missing-Problem)
       - [Joined Limbs Into Body Information](#Joined-Limbs-Into-Body-Information)
     - [Human Parser Influence](#Human-Parser-Influence)
   - [User Guide](#User-Guide)
     - [Training](#Training)
     - [Testing](#Testing)


## Dataset
- Based on [Toward Characteristic-Preserving Image-based Virtual Try-On Network](https://github.com/sergeywong/cp-vton)(CP-VTON) dataset.
- Used [Self Correction for Human Parsing](https://github.com/PeikeLi/Self-Correction-Human-Parsing) to achieve better parsing results.
- Added male model images extracted from [FashionGen dataset](https://fashion-gen.com) to encompass all genders for real world usage.

## Discussion
- We replicated [CP-VTON](https://github.com/sergeywong/cp-vton)'s Try-On Module by the implementation details their paper provided to make comparison with our cycleTryOn module in this section.

### Geometric Matching Limitation
- Although Geometric Matching Module(GMM), proposed by CP-VTON, had proven its efficiency in aligning in-shop clothes with human image, GMM did not have the ability to differentiate inside and outside of clothes, To be more precisely, GMM tends to force the WHOLE in-shop cloth into the original cloth shape on person if the deformation grid was dense enough.

- **Cycle Consistency loss** and **Adversarial loss** were introduced to calibrate this kind of unrealistic problems in this work. 

- As shown in Fig.2, the lining and tag of target clothes still existed in results of CP-VTON. In comparison to CP-VTON, our cycleTryOn module had learned how to hide the inner part of clothes and successfully generated the appropriate skin color of the person.

<div align="center">
 <img src="image/GML.png" width="700px" />
 <p>Fig.2 In contrary to the lining showing problem with CP-VTON's, our cycleTryOn module had successfully adjusted this problem.</p>
</div>

### Arms Missing Problem
- For some rare pose in dataset, such as pose with folded arms, failure rate become extremely high in CP-VTON. In fact there is NO succese testing result in our reproduced CP-VTON model. 
- In our perspective, networks without inverse mapping is hard to learn in which case people's body information should be reserved. The best strategy of generator is paste the warped cloth on the right position when the data of folded arms pose is short.
- **Cycle Consistency** take a huge advantage to minimize this problem. It's simply because if people's body information is missing in final result, the load of generator to mapping back to original image becomes too heave, so the best strategy will be reserve those information at the first time.

<div align="center">
 <img src="image/AMP.png" width="700px" />
 <p>Fig.3 In comparison of arms missing problem with CP-VTON's results, our cycleTryOn module successfully reserve arms information. Even though our model haven't optimize to its best state, the tendency is still obvious.</p>
</div>

#### Joined Limbs Into Body Information
- After solve arms missing problem we joined limbs information into body information aims to refine the details.
- Provides limbs information to the network do have great limbs details in result as shown in Fig.4. But the presence of limbs information in input seems to strongly limit the cloth shape to the original one on model(see Fig.4).
- Although the usage of providing limbs information is restricted to same cloth type only, we still suggest training model by case for real word usage cause the refinement of details is too good.

<div align="center">
 <img src="image/LE1.png" width="700px" />
 <p>Fig.4 Comparison of limbs information been joined.</p>
 <img src="image/LE2.png" width="700px" />
 <p>Fig.5 Cases of the cloth type changed.</p>
</div>

### Human Parser Influence
- Since GMM takes the original cloth on person as ground truth by using human parser to crop the cloth out from model images, we consider the higher mIoU human parsing network can achieve the better GMM result would be.

<div align="center">
 <img src="image/PI.png" width="350px" />
 <p>Fig.6 Higher IoU dose improve GMM result .</p>
</div>

## User Guide
- The framework of Try-On Module in this project refer to [Virtually Trying on New Clothing with Arbitrary Poses](https://www.english.com.tw/modules/newbb/viewtopic.php?post_id=928), but change the training strategy and pipeline for different purpose.
### Training
- example command, ```--uselimbs``` is an option for certain Try-On task, please see [Joined Limbs Into Body Information](#Joined-Limbs-Into-Body-Information).
```
python train_cycleTryOn.py --name 'gmm_train' --stage 'GMM' --save_count 5000 --shuffle
python train_cycleTryOn.py --name 'cycleTryOn_train' --stage 'cycleTryOn' --uselimbs
```
### Testing
```
python test_cycleTryOn.py --name 'gmm_test' --stage 'GMM' --datamode test --data_list 'test_pairs.txt' --checkpoint checkpoints/gmm_train/gmm_final.pth
python test_cycleTryOn.py --name 'cycleTryOn_test' --stage 'cycleTryOn' --uselimbs --datamode test --data_list 'test_pairs.txt' --checkpoint checkpoints/cycleTryOn_train/cycleTryOn_final.pth
```
## Reference
- Wang, Bochao, et al. "Toward characteristic-preserving image-based virtual try-on network." Proceedings of the European Conference on Computer Vision (ECCV). 2018.
- Zheng, Na, et al. "Virtually trying on new clothing with arbitrary poses." Proceedings of the 27th ACM International Conference on Multimedia. 2019.
