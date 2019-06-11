# TrackR-CNN
Code for the TrackR-CNN baseline for the Multi Object Tracking and Segmentation (MOTS) task.

## Project website (including annotations)
https://www.vision.rwth-aachen.de/page/mots

## Paper
https://www.vision.rwth-aachen.de/media/papers/mots-multi-object-tracking-and-segmentation/MOTS.pdf

## mots_tools for evaluating results
https://github.com/VisualComputingInstitute/mots_tools

## Running this code
### Setup
You'll need to install the following packages (possibly more):
```
tensorflow-gpu pycocotools numpy scipy sklearn pypng opencv-python munkres
```
Also create the following directories for logs, model files etc.:
```
mkdir forwarded models summaries logs
```

### Training
In order to train a model, run `main.py` with the corresponding configuration file. For the baseline model with two separable 3D convolutions and data association with learned embeddings, use
```
python main.py configs/conv3d_sep2
```
You'll need to adjust the `KITTI_segtrack_data_dir` and `load_init` flags to point to the KITTI MOTS data directory and the path to the pretrained model, respectively. Logs, checkpoints and summaries are stored in the `logs/`, `models/` and `summaries/` subdirectories.

### Forwarding and tracking
To obtain the model's predictions (we call this "forwarding") run:
```
python main.py configs/conv3d_sep2 "{\"task\":\"forward_tracking\",\"dataset\":\"KITTI_segtrack_feed\",\"load_epoch_no\":5,\"batch_size\":5,\"export_detections\":true,\"do_tracking\":false,\"video_tags_to_load\":[\"0002\",\"0006\",\"0007\",\"0008\",\"0010\",\"0013\",\"0014\",\"0016\",\"0018\",\"0000\",\"0001\",\"0003\",\"0004\",\"0005\",\"0009\",\"0011\",\"0012\",\"0015\",\"0017\",\"0019\",\"0020\"]}"
```
The json string supplied as an additional argument here overwrites the settings in the config file. Use `video_tags_to_load` to obtain predictions for specific sequences (in the example, all KITTI MOTS sequences are chosen). Output is written to the `forwarded/` subdirectory.

The model predictions as obtained by the previous command are not yet linked over time. You can use the following command to run the tracking algorithm described in the paper and to obtain final results in the `forwarded/` subdirectory which can be processed by the mots_tools scripts:
```
python main.py configs/conv3d_sep2
"{\"build_networks\":false,\"import_detections\":true,\"task\":\"forward_tracking\",\"dataset\":\"KITTI_segtrack_feed\",\"do_tracking\":true,\"visualize_detections\":false,\"visualize_tracks\":false,\"load_epoch_no\":5,\"video_tags_to_load\":[\"0002\",\"0006\",\"0007\",\"0008\",\"0010\",\"0013\",\"0014\",\"0016\",\"0018\"]}"
```
You can also visualize the tracking results here by setting `visualize_tracks` to true.

### Tuning
The script for random tuning will find the best combination of tracking parameters on the training set and then evaluate these parameters on the validation set. This is how the results in the MOTS paper are obtained.

To use this script, run
```
python scripts/eval/segtrack_tune_experiment.py /path/to/detections/ /path/to/groundtruth/ /path/to/precomputed_optical_flow/ /path/to/output_file /path/to/tmp_folder/ /path/to/mots_eval/ association_type num_iterations
```
where `/path/to/detections/` is a folder containing the model output on the training set (obtained by the forwarding command above); `/path/to/mots_eval/` refers to the official evaluation script (link see above); `association_type` determines the method for associating detections into tracks and is either `reid` (using the association embeddings), `mask` (using mask warping), `bbox_iou` (using bounding box warping with median optical flow) or `bbox_center` (nearest neighbor matching); `num_iterations` is the number of random trials (1000 in the paper).

## References
Parts of this code are based on Tensorpack (https://github.com/tensorpack/tensorpack/tree/master/examples/FasterRCNN) and RETURNN (https://github.com/rwth-i6/returnn/blob/master/Log.py).

## Citation
If you use this code, please cite:
```
@inproceedings{Voigtlaender19CVPR_MOTS,
 author = {Paul Voigtlaender and Michael Krause and Aljo\u{s}a O\u{s}ep and Jonathon Luiten and Berin Balachandar Gnana Sekar and Andreas Geiger and Bastian Leibe},
 title = {{MOTS}: Multi-Object Tracking and Segmentation},
 booktitle = {CVPR},
 year = {2019},
}
```

## License
MIT License
