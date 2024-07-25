<p align="center">

  <h1 align="center">Bridge the gap between 3D reconstruction and industry pipeline</h1>


<p align="center">
Using 2dgs and GOF to extract mesh from 3D gaussians in glTF 2.0 standard</p>
<br>


# TO DO
- [x] Make it a one pass pipeline and do not need to save the intermediate results
- [x] Normal map still needs improvement, please refer to cuda_rasterizer/forward.cu



# Installation
Clone the repository and create an anaconda environment using
```

# Please nofitify that I am using the Nvidia 3080Ti GPU
# If you are using a different GPU, you might need to change the version of torch and torchvision
pip install torch==2.3.1 torchvision==0.18.1 torchaudio==2.3.1 --index-url https://download.pytorch.org/whl/cu121

pip install -r requirements.txt

pip install submodules/diff-gaussian-rasterization
pip install submodules/simple-knn/

# tetra-nerf for triangulation
cd submodules/tetra-triangulation
cmake -S . -B build
# build the project from Visual Studio or using the following command
cmake --build build --config Release

```

# Dataset

You need to download the ground truth point clouds from the [DTU dataset](https://roboimagedata.compute.dtu.dk/?page_id=36) and save to `dtu_eval/Offical_DTU_Dataset` to evaluate the geometry reconstruction. For the [Tanks and Temples](https://www.tanksandtemples.org/download/) dataset, you need to download the ground truth point clouds, alignments and cropfiles and save to `eval_tnt/TrainingSet`, such as `eval_tnt/TrainingSet/Caterpillar/Caterpillar.ply`.


# Custom Dataset
We use the same data format from 3DGS, please follow [here](https://github.com/graphdeco-inria/gaussian-splatting?tab=readme-ov-file#processing-your-own-scenes) to prepare the your dataset. Then you can train your model and extract a mesh (we use the Tanks and Temples dataset for example)
```
# training
# --use_decoupled_appearance to enable decoupled appearance modeling if your images has changing lighting conditions
python train.py -s playroom -m playroom/output -r 2 --use_decoupled_appearance

# extract the mesh after training
python extract_mesh.py -m playroom/output --iteration 30000

```

# Acknowledgements && Citations
This project is built upon [3DGS](https://github.com/graphdeco-inria/gaussian-splatting), [GOF](https://github.com/autonomousvision/gaussian-opacity-fields) and [2DGS](https://surfsplatting.github.io/). Regularizations and some visualizations are taken from [2DGS](https://surfsplatting.github.io/). Tetrahedra triangulation is taken from [Tetra-NeRF](https://github.com/jkulhanek/tetra-nerf). Marching Tetrahdedra is adapted from [Kaolin](https://github.com/NVIDIAGameWorks/kaolin/blob/master/kaolin/ops/conversions/tetmesh.py) Library. We thank all the authors for their great work and repos. 


