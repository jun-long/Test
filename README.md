import cv2
import numpy as np

def is_blue_or_white_more(img, box):
    """
    判斷在給定的 box 區域中，藍色和白色哪個顏色較多。
    
    :param img: 原始圖片
    :param box: YOLO 的 box 格式：[x_min, y_min, x_max, y_max]
    :return: 'blue' 或 'white' 表示哪種顏色占比大於一半
    """
    x_min, y_min, x_max, y_max = box
    # 擷取 box 區域
    roi = img[y_min:y_max, x_min:x_max]
    
    # 定義藍色和白色的 HSV 範圍
    blue_lower = np.array([100, 150, 0])
    blue_upper = np.array([140, 255, 255])
    
    white_lower = np.array([0, 0, 200])
    white_upper = np.array([180, 20, 255])

    # 將 RGB 轉換為 HSV
    hsv = cv2.cvtColor(roi, cv2.COLOR_BGR2HSV)

    # 藍色遮罩
    blue_mask = cv2.inRange(hsv, blue_lower, blue_upper)
    blue_pixels = cv2.countNonZero(blue_mask)

    # 白色遮罩
    white_mask = cv2.inRange(hsv, white_lower, white_upper)
    white_pixels = cv2.countNonZero(white_mask)

    # 計算藍色和白色像素比例
    total_pixels = roi.shape[0] * roi.shape[1]
    blue_ratio = blue_pixels / total_pixels
    white_ratio = white_pixels / total_pixels

    # 比較比例
    if blue_ratio > white_ratio:
        return 'blue'
    else:
        return 'white'

# 主程式
def main(image_path, boxes):
    """
    處理圖片並比較每個 box 的顏色占比。
    
    :param image_path: 圖片的路徑
    :param boxes: 包含 YOLO 格式 box 的清單，每個 box 格式：[x_min, y_min, x_max, y_max]
    """
    img = cv2.imread(image_path)

    # 逐一處理每個 box
    for i, box in enumerate(boxes):
        color = is_blue_or_white_more(img, box)
        print(f"Box {i + 1}: {color} 色較多")

# 測試範例
image_path = 'path/to/your/image.jpg'  # 替換成你的圖片路徑
boxes = [
    [50, 100, 200, 300],   # 替換成你要的 box 座標
    [250, 150, 400, 350]
]
main(image_path, boxes)
