---
title: "bootstrap your own latent: a new approach to self-supervised learning : BYOL에서 predictor를 쓰는 이유와 실험을 하게된 motivation에 대한 개인적인 생각"
date: 2021-04-11 05:55:28 -0400
categories: contrastive learning
---


우선 BYOL이 이러한 방식을 실험하게된 동기는 Self-organizing neural network that discovers surfaces in random-dot stereograms 이라는 논문에서 시작한다. 이 논문은 하나의 이미지에 두 개의 네트워크를 사용하였을 때 나오는 두개의 inference의 squared difference를 줄이는 방법으로 representation을 배울 수 있는 방법을 먼저 설명한다. 그러나 이 방법으로 training 하였을 때, 결국 networks는 collapsed representation을 얻게 되었다고 한다. 여기서 collapsed representation을 얻는다는 것은 두개의 networks의 output이 같은 constant vector를 내어놓는 것으로 squared difference는 0으로 수렴하여 model이 느끼기에는 loss가 0으로 수렴하여 좋은 representation을 얻게 된다고 생각하지만 실제로는 어떤 이미지를 input으로 하느냐에 관계없이 constant vector만 output으로 내놓는 의미 없는 representation을 얻게되는 것을 말한다.

이 논문에서 이를 해결하기 위해 networks의 ensemble을 사용한다.


![image](https://user-images.githubusercontent.com/70498916/114298059-5a6ebc80-9aef-11eb-84db-2237422c09e1.png)



다음 그림과 같이 여러개의 network의 output을 서로 다른 weight으로 합해진 output과 독립적인 하나의 network의 output의 squared difference를 줄이는 방식으로 similarity를 측정하여 collapsed representation을 피할 수 있다고 주장한다.

이 논문에서 동기를 얻은 후 실험을 하나 하였는데 실험의 목적은 과연 randomly init network의 output을 target으로 배워도 정확도 개선이 있을까에 초점을 맞춘 것이다.

이 실험은 다른 블로그에서 그림으로 자세히 설명이 되어있다.

여튼 이러한 실험을 통해 randomly init network의 output을 target으로 쓰면 정확도가 향상된다는 결과와 위의 논문에서 networks의 outputs의 ensemble을 통해 collapsed representation을 피할 수 있다는 intuition을 얻어서 model에 predictor를 추가한 것이다. 이때 predictor는 MLP의 구조를 하고 있는데 이 MLP의 구조가 network ensemble의 역할을 하는 것이다.(MLP 이전의 output의 각 원소들이 weighted sum으로 다음 layer의 output이 되므로)

그리고 위에서 한 실험은 MLP를 target을 만들어 주는 network에 붙이고 byol은 online network, 즉 target을 배우는 쪽에 붙이는 데, 이는 online network만 gradient descent를 하기 때문이다. target network에 MLP를 붙이면 gradient descent가 안되므로 결국 random init한 상태 그대로 output을 내놓기 때문이다.

여담으로 이러한 collapsed representation은 SimCLR나 MoCo에서는 negative pair를 사용하여 해결하고 있다. positive pair와의 similarity 뿐만 아니라 negative pair와도 거리가 멀어져야 하므로 collapsed representation이 생기지 않기 때문이다.


