Proposed: December 2, 2021

Status: Passed

Link: [GitHub PR](https://github.com/BeanstalkFarms/Beanstalk/pull/14), [Arweave](https://arweave.net/31Mo38aJmBIMxH47fS3YKOK97wuHxrbsev7x7lUBDfc)

## Proposer

Beanstalk Farms

## Summary

- Change the Minimum Soil Rate to factor in the Weather and Pods harvested at the start of the Season.
- Have the Soil decrease by the time weighted average shortage of Beans in the BEAN:ETH pool over the previous Season $\Delta \bar{b}_{t-1}$.

## Problem

Currently, the Available Soil is at or below the Minimum Soil Rate every Season. Based on the current Weather (~2000%+) and current Minimum Soil Rate (.1%), Beanstalk is willing to issue Pods worth more than 2% of the total Bean supply every hour. While the Pod Rate has started to decrease slightly, when there is excess demand for Soil and Beans, Beanstalk should be deleveraging faster than it currently is. This is indicative that the Minimum Soil Rate is implemented in a suboptimal fashion: it does not currently factor in the Weather $w$ or Pods being Harvested at the start of each Season.

Having observed the behavior of lenders during the previous few weeks, it is evident that scarcity of Soil is a large driver of demand for Soil: As Soil started to diminish, there was a race for Soil. Increasing the scarcity of Soil when ![](https://user-images.githubusercontent.com/9626499/196241696-9f30fdce-aa34-4d9b-bf61-2a27abdd468d.png) should further improve the efficiency of the Soil market. When ![](https://user-images.githubusercontent.com/9626499/196241696-9f30fdce-aa34-4d9b-bf61-2a27abdd468d.png) for a Season, the Soil currently does not decrease even though Beanstalk does not need to attract as many lenders as when $\bar{P}_{t-1} < 1$. 

## Proposed Solution

Change the Minimum Soil Rate to factor in the Weather and Pods harvested at the start of the Season. The following logic will be added/updated to the whitepaper to reflect the following:

We define the number of newly Harvestable Pods at the start of Season $t$ $d^h_t$ such that $d^h_t \in \{j \times 10^{-6} | j \in N\}$ as:

$$d_t^h = \text{min}\left(\text{max}\left(\frac{\Delta\bar{b}_{t-1}}{2},\ 0\right),\ D\right)$$

Recalling $S^{\text{min}}_t$, $S_t^{\text{max}}$ are the minimum and maximum soil at Season $t$ respectively, and $S^{\text{start}}_t$, $S^{\text{end}}_t$ are the Soil at the start and end of Season $t$, respectively, the formulas

![](https://user-images.githubusercontent.com/9626499/196239715-6f00559b-931d-4109-95e1-3a576b6a7432.png)

![](https://user-images.githubusercontent.com/9626499/196239536-ed97226b-6349-441f-9e9a-c09c38133b56.png)

![](https://user-images.githubusercontent.com/9626499/196239791-e69106c0-c472-4f0d-bf5e-6aa31bde442d.png)

Note, the current formula for $S_t^{\text{start}}$ in the whitepaper is actually the correct formula for the new implementation of Soil. For clarity, the old formula for $S_t^{\text{start}}$ should have been: 

$$S_t^{\text{start}} = \text{min}(\text{max}(S_{t-1}^{\text{end}} - \text{min}(0,\ \Delta \bar{b}_{t-1}),\ S_t^{\text{min}}),\ S_t^{\text{max}})$$

Furthermore, the complex demand for Soil will continue to be triggered when the Available Soil is less than .1% of the total Bean supply. It will no longer be dependent on the Minimum Soil Rate. Going forward, it will be dependent on the ComplexSoilRate, which will remain .1% 

## Economic Rationale

Beanstalk will now have the following invariant when operating at minimum Soil:

$$S^{\text{start}}_t(1 + \frac{w}{100}) = d^h_t$$

This means that the Pod line is at most going to stay the same length when there is excess demand for Soil and $\bar{P}_{t-1} > 1$. If all Soil was sown in the previous Season, then the Pod line will remain the same length. If less than all of the Soil was sown, the Pod line will decrease.

By setting the Soil so that in instances where all Soil is sown the Pod Line stays the same length Beanstalk will maintain the same amount of debt, while increasing the supply. This will cause the Pod Rate to decrease quickly during instances where there is excess demand for Soil, but in a way that does not allow Beanstalk to easily run out of outstanding debt, except in the instance of a Season of Plenty. 

By decreasing the outstanding Soil when ![](https://user-images.githubusercontent.com/9626499/196239953-a8ab00f0-4695-4222-9548-983496a0a398.png), there is more likely to be an efficient market for Soil when ![](https://user-images.githubusercontent.com/9626499/196240030-bf801d3c-3e3a-4861-8d40-49ba29863ed0.png) because it is no longer a reasonable expectation that there will be excess Soil when $\bar{P}_{t-1} > 1$. A more efficient Soil market will lower the average Weather Beanstalk has to pay over time, and improve the ability of Beanstalk to regularly cross the Bean price over its Peg. 

## Effective

Effective immediately upon commit.