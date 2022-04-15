# MCMICROGUI: Interactive portable reporting and visualization for MCMICRO

## Abstract:

MCMICRO is a is a modular image processing pipeline for multiplexed tissue images that allows for reproducible analysis of both Tissue Microarray (TMA) and whole slide images (WSI) with minimal manual preprocessing<sup>5</sup>. However, MCMICRO provides no built in quality assurance and visualization package. Here, we present an early release of MCMICRO-GUI: a lightweight application capable of generating automatic, interactive and portable reports from data processed with MCMICRO.

The source code the application can be found on [https://github.com/SchapiroLabor/mcmicrogui](https://github.com/SchapiroLabor/mcmicrogui), alongside with instructions on how to run the software. A containerized version is also available under [https://github.com/SchapiroLabor/mcmicrogui/pkgs/container/mcmicrogui](https://github.com/SchapiroLabor/mcmicrogui/pkgs/container/mcmicrogui).


## 1. Introduction: Multiplexed image analysis with MCMICRO

Multiplexed tissue imaging allows for precise molecular analysis of individual cells in the spatial context of their tissue. It enables quantification of up to 100 different proteins at subcellular resolutions across large tissue specimens. Thus it can be used to reveal valuable new insights about disease pathogenesis, tissue architecture and cell-cell interactions. Furthermore it is a promising tool for augmenting both transcriptomics<sup>1–3</sup> as well as traditional histopathological methods of diagnosis<sup>2,4</sup>.

Since multiplexed imaging can yield up to 1 terabyte of data, accurate and reproducible analysis of these large datasets poses a number of computational issues5. Image processing needs to be efficient in terms of memory and processing power yet precise enough to allow for the detection and segmentation of individual cells in a crowded environment<sup>5</sup>.

In 2022, Schapiro et.al. presented MCMICRO (Multiple Choice MICROscopy) as a solution for these issues. The software is a modular image processing pipeline that allows for reproducible analysis of both Tissue Microarray (TMA) and whole slide images (WSI) with minimal manual preprocessing<sup>5</sup>.

MCMICRO outputs the result of its analysis in the form of images and tabular data. Thus it complements existing software and gives the user flexibility in terms of further analysis and visualization<sup>5</sup>. This approach is very useful in the context of a bioinformatics laboratory where tools and know-how for these tasks are readily available.
 However, the insights that can be gained from processing data with MCMICRO would also be useful in a wet lab or clinical setting. Furthermore, being able to conduct quality assurance of an MCMICRO analysis without time consuming further analysis would also be highly desireable for bioinfomaticians.

In order to solve these issues and to expand the MCMICRO ecosystem, we set out to develop MCMICRO-GUI: a lightweight application capable of generating automatic, interactive and portable reports from data processed with MCMICRO.


## 2. Methods



### 2.1 Design Goals

At the beginning of development we compiled the following list of possible features the ideal MCMICRO-GUI application should have:

- Should display basic information about an MCMICRO analysis, such as the number of detected cells or the number of cores for a TMA
- Should allow the user to view the processed image data by offering both a low resolution overview image and a high resolution sample
- Should allow the user to easily identify TMA cores on a TMA and visualize the order in which they were processed by MCMICRO
- Should allow the user to assess the quality of cell and nuclear segmentation
- Should offer the ability to compare the results of multiple segmentation attempts
- Should offer first analytical insights by displaying a number of chosen markers and offering TSNEs and UMAPs based on the processed data

Based on this list of features, we specified the following four design goals for the tool and the resulting visualization:

- Interactivity - the visualization should allow the user to get an quick interactive overview over his MCMICRO run
- Accessibility - viewing the results should require little to no technical knowledge
- Portability - the visualization should come in a portable format, to facilitate sharing results
- Expandability - the tool should be easily expandable in terms of features and should integrate well with MCMICRO

Two possible ways of achieving these goals were discussed:

- Building MCMICRO-GUI as fully featured web application that processes MCMICRO data in real time
- Building a program that generates a standalone report file from MCMICRO data

While a web application would allow for a more interactive visualization, installing and setting up such an application can be time consuming and even frustrating for end users with limited technical knowledge.
 To increase portability and accessibility we therefore decided for the second approach.


### 2.2 Computational tools and packages

For processing and parsing the data generated by MCMICRO, we used Python. It allows for rapid prototyping and offers a number of helpful packages that were used in the development process:

- Scikit-image for reading and labelling images
- Numpy for performing calculations on images
- Plotly-python for generating interactive plots and images based on the data parsed
- Jinja for rendering html with external data based on a template

For visualization and GUI purposes we used native html and Javascript without any external libraries. To containerize the application for improved portability and better integration into the MCMICRO pipeline, we used Docker and Singularity. Git and Github were used for version control management.


## 3. Results

We developed MCMICRO-GUI as a tool consisting of two components. The first component is an interactive report using JavaScript and html to visualize important aspects of an MCMICRO analysis. This report is generated from data processed with MCMICRO by a command line application implemented in Python. 

<br/><br/>
![alt text](https://github.com/moritz-neesen/mcmicrogui-report/blob/main/UML.png)

_Fig. 1: UML flowchart of MCMICRO-GUI_
<br/><br/>


When run, the application follows the steps outlined in Figure 1.

In the first step, the output of an MCMICRO analysis is read from a path specified by the user. The program then automatically determines from the folder structure if the data it is processing is derived from a TMA.

If no TMA is present, MCMICRO-GUI then detects a high intensity spot on the image created by MCMICROs registration step. A 500x500 pixel region around this spot is then cropped from both the registration image and the cell segmentation mask provided by MCMICRO. Both outcrops are stored in memory and are used to visualize image and segmentation data later on.

Next, all images and masks are downscaled in proportion to their original size and the high resolution outcrop is overlayed with the cropped cell segmentation mask. The original image data is preserved in this process.

The images are then plotted as plotly plots and inserted into the prepared html template via Jinjas templating engine. Finally, the result is saved as an html file.

If a TMA is detected, an additional step is required before the report can be generated. Since MCMICRO processes and stores each core individually, the individual masks for the single cores are inserted into an empty image at their respective centroids. The result is a core mask and a segmentation mask of the size of the whole TMA, which is then cropped and scaled in the same way as a WSI. This allows the user to view their TMA cores and segmentation results in the context of the whole image.

As of version 0.1.0, the interactive report offers three pages with data and zoomable image plots.

The first page serves as general overview. Here the user can view their image and a high resolution sample as well as some basic information about their data.

<br/><br/>
![alt text](https://github.com/moritz-neesen/mcmicrogui-report/blob/main/Overview.png)

_Fig. 2: Visualization of basic MCMICRO data_
<br/><br/>


On the second page, information about TMA cores is displayed. Hovering over a TMA core presents various data points and the legend can be used to highlight individual cores. If the dataset was a non-TMA dataset, this page displays a message that no TMA data could be found.

<br/><br/>
![](https://github.com/moritz-neesen/mcmicrogui-report/blob/main/TMA.png)

_Fig. 3: TMA core visualization_
<br/><br/>


The third page offers a preview of MCMICRO&#39;s cell segmentation, using the same high resolution sample as the first page. To be able to easily assess the quality of the segmentation, each cell is plotted and highlighted as trace that can be hovered over.


<br/><br/>
![](https://github.com/moritz-neesen/mcmicrogui-report/blob/main/Segmentation.png)

_Fig. 4: Segmentation preview_
<br/><br/>

## 4. Discussion

Despite the short development time of eight weeks, we managed to build a fully functional application that includes four out of the six features we brainstormed during the concept phase. Except for some issues with containerizing the application, we encountered no major hurdles during development. Our approach of generating an offline html document to visualize data from MCMICRO allowed us to meet most of our initial design goals in a reasonable amount of time.

However, MCMICRO-GUI still has a few limitations and room for new features in the future, which is why we consider the current version a pre-release version.

For future versions we are planning to add more customization options and more advanced analysis features. Allowing the user to compare multiple segmentation algorithms or including previews for multiple customizable makers would increase the usefulness of the application.

With large datasets, the html report also occasionally suffers from performance issues. This is most likely caused by using plotly in an offline html, as several thousand plot traces have to be rendered by local JavaScript.

Nevertheless, we consider MCMICRO-GUI an important addition to the MCMICRO ecosystem. With our tool being able to visualize results quickly, it is likely to speed up working with MCMICRO considerably and thus we hope to see it become an official part of the MCMICRO pipeline in the near future.

## 5. References

1. Vickovic, S. _et al._ High-definition spatial transcriptomics for in situ tissue profiling. _Nat. Methods_ **16** , 987–990 (2019).

2. Chen, K. H., Boettiger, A. N., Moffitt, J. R., Wang, S. &amp; Zhuang, X. RNA imaging. Spatially resolved, highly multiplexed RNA profiling in single cells. _Science_ **348** , aaa6090 (2015).

3. Rodriques, S. G. _et al._ Slide-seq: A scalable technology for measuring genome-wide expression at high spatial resolution. _Science_ **363** , 1463–1467 (2019).

4. Keren, L. _et al._ A Structured Tumor-Immune Microenvironment in Triple Negative Breast Cancer Revealed by Multiplexed Ion Beam Imaging. _Cell_ **174** , 1373-1387.e19 (2018).

5. Schapiro, D. _et al._ MCMICRO: a scalable, modular image-processing pipeline for multiplexed tissue imaging. _Nat. Methods_ **19** , 311–315 (2022).
