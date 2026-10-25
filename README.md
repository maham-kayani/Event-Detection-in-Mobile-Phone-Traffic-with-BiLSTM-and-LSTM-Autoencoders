# Unsupervised Event Detection in Mobile-Phone Traffic with LSTM and BiLSTM Autoencoders

Code and geospatial results for a three-city study of event detection in mobile-phone traffic in Paris, Shanghai and Milan.

Events such as public holidays, football matches or a fire are rare and are seldom labelled in mobile data. We train LSTM and BiLSTM autoencoders on ordinary days only, and score each day by how far it lies from an **expected day** of the same type. This also works for quiet days such as public holidays, where the usual reconstruction error fails.

## What is in this repository

| Folder | Content |
|---|---|
| `geospatial/` | Day-by-day geospatial files of the analysis for the three cities |
| `figures/` | Figures used in the paper |



## Geospatial files

For every day and every area, the geospatial files show what the system detects.

- **Content:** the deviation of each grid square from the expected day, and the city and local scores.
- **Cities:** Paris, Shanghai and Milan.
- **Format:** write the real format here, for example GeoJSON, GeoPackage or HTML maps.
- **How to open:** drag the files into QGIS, or open the HTML maps in any browser.

## Datasets

The raw data is **not included** in this repository. Please download it from the original sources and follow their terms of use.

### Paris: NetMob23

NetMob23 is a high-resolution, multi-region dataset of service-level mobile traffic, described by Martínez-Durive et al. in 2023. We use the Paris part.

| Item | Value |
|---|---|
| Period | 1 to 30 April 2019, which is 30 days |
| Spatial unit | 100 m tiles, with 10,374 inside Paris |
| Time resolution | 15 minutes |
| Volume | 16.0 billion records in 2,040 daily files |
| Content | Uplink and downlink traffic of 68 mobile applications |

How we use it:
- Tiles inside Paris are grouped into 250 m squares. Squares with fewer than four tiles are dropped, which leaves 1,670 squares.
- The 15-minute values are summed into hours, and the 68 applications are added into two channels: uplink and downlink.
- The traffic of each application is also kept, to see which applications react to an event.
- NetMob23 is available for research purposes only. Please request access and respect its licence. Do not redistribute the raw data.

Paper: O. E. Martínez-Durive, S. Mishra, C. Ziemlicki, S. Rubrichi, Z. Smoreda, M. Fiore, "The NetMob23 dataset: A high-resolution multi-region service-level mobile data traffic cartography", 2023. https://arxiv.org/abs/2305.06933

### Shanghai: Shanghai Telecom

Internet sessions from 3,042 base stations between 1 June and 30 November 2014, with about 9,739 users. Base stations are placed on a 1 km grid, and there are two signals per square and hour: distinct users and connection minutes.

Papers cited by the dataset providers:
1. Y. Li, A. Zhou, X. Ma, S. Wang, "Profit-aware Edge Server Placement", IEEE Internet of Things Journal, vol. 9, no. 1, pp. 55-67, 2022. PDF: https://wangshangguang.github.io/assets/IOTJLI.pdf Source code: https://wangshangguang.github.io/files/ESP.zip
2. Y. Guo, S. Wang, A. Zhou, J. Xu, J. Yuan, C. Hsu, "User Allocation-aware Edge Cloud Placement in Mobile Edge Computing", Software: Practice and Experience, vol. 50, no. 5, pp. 489-502, 2020. PDF: https://wangshangguang.github.io/assets/spe.2685.pdf Source code: https://wangshangguang.github.io/files/edge-cloud-placement.rar
3. S. Wang, Y. Guo, N. Zhang, P. Yang, A. Zhou, X. Shen, "Delay-aware Microservice Coordination in Mobile Edge Computing: A Reinforcement Learning Approach", IEEE Transactions on Mobile Computing, vol. 20, no. 3, pp. 939-953, 2021. PDF: https://wangshangguang.github.io/assets/Delay-aware_Microservice_Coordination.pdf

### Milan: Telecom Italia

SMS, call and internet activity on a 100 x 100 grid of about 235 m squares, from 1 November 2013 to 1 January 2014, with 11 to 20 December missing. There are five channels.

- Dataset: Telecom Italia, "Telecommunications - SMS, Call, Internet - MI", Harvard Dataverse, 2015. https://doi.org/10.7910/DVN/EGZHFV
- Paper: G. Barlacchi et al., "A multi-source dataset of urban life in the city of Milan and the Province of Trentino", Scientific Data, vol. 2, 150055, 2015. https://doi.org/10.1038/sdata.2015.55

## Method in short

1. **Preprocessing:** grid squares, hourly values, the logarithm of 1 plus x, and per-square standardization.
2. **Samples:** one square on one day, 24 hours times the number of channels, plus a day type that is either working day or rest day.
3. **Autoencoder:** LSTM or BiLSTM, trained on ordinary days only, with a latent vector of 16 values.
4. **Scoring:** the expected day is decoded from the median latent vector of the ordinary days of the same type. The deviation of the rebuilt day from the expected day gives:
   - a **city score**, which is the negative mean deviation over all squares and finds quiet days such as holidays;
   - a **local score**, which is the mean deviation over the 3 x 3 squares around a site at the busiest hour and finds local events.
5. **Ranking:** days are ranked by score. An event is found if its day is in the top 10%.
6. **Evaluation:** district-based split with a 1 km buffer, compared with a simple baseline that uses the median of the same weekday within 28 days.

## Citation

If you use the geospatial files, please cite our paper and the original datasets. Add the final reference of our paper here.

**Paris: NetMob23**

```bibtex
@misc{netmob23,
  author        = {Mart{\'i}nez-Durive, Orlando E. and Mishra, Sachit and Ziemlicki, Cezary and Rubrichi, Stefania and Smoreda, Zbigniew and Fiore, Marco},
  title         = {The {NetMob23} dataset: A high-resolution multi-region service-level mobile data traffic cartography},
  year          = {2023},
  eprint        = {2305.06933},
  archivePrefix = {arXiv},
  primaryClass  = {cs.NI}
}
```

**Milan: Telecom Italia**

```bibtex
@misc{telecomitalia2015,
  author    = {{Telecom Italia}},
  title     = {Telecommunications - {SMS}, Call, Internet - {MI}},
  year      = {2015},
  publisher = {Harvard Dataverse},
  version   = {V1},
  doi       = {10.7910/DVN/EGZHFV},
  url       = {https://doi.org/10.7910/DVN/EGZHFV}
}

@article{barlacchi2015multi,
  author  = {Barlacchi, Gianni and others},
  title   = {A Multi-source Dataset of Urban Life in the City of {Milan} and the {Province of Trentino}},
  journal = {Scientific Data},
  volume  = {2},
  pages   = {150055},
  year    = {2015},
  doi     = {10.1038/sdata.2015.55}
}
```

**Shanghai: Shanghai Telecom**

```bibtex
@article{li2022profit,
  author  = {Li, Yuanzhe and Zhou, Ao and Ma, Xiao and Wang, Shangguang},
  title   = {Profit-aware Edge Server Placement},
  journal = {IEEE Internet of Things Journal},
  volume  = {9},
  number  = {1},
  pages   = {55--67},
  year    = {2022}
}

@article{guo2020user,
  author  = {Guo, Yu and Wang, Shangguang and Zhou, Ao and Xu, Jinliang and Yuan, Jie and Hsu, Ching-Hsien},
  title   = {User Allocation-aware Edge Cloud Placement in Mobile Edge Computing},
  journal = {Software: Practice and Experience},
  volume  = {50},
  number  = {5},
  pages   = {489--502},
  year    = {2020}
}

@article{wang2021delay,
  author  = {Wang, Shangguang and Guo, Yu and Zhang, Ning and Yang, Peng and Zhou, Ao and Shen, Xuemin},
  title   = {Delay-aware Microservice Coordination in Mobile Edge Computing: A Reinforcement Learning Approach},
  journal = {IEEE Transactions on Mobile Computing},
  volume  = {20},
  number  = {3},
  pages   = {939--953},
  year    = {2021}
}
```

## Licence

Add your licence here, for example MIT for the code. The datasets keep their own licences.
