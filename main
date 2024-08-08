//Author: Ruben Rehal
//Date: July 17, 2024

#include <opencv2/opencv.hpp>
#include <iostream>

using namespace cv;
using namespace std;

// Function to detect and track the hand
void trackHand(Mat& frame) {
    Mat hsvImage, skinMask;
    
    // Convert the frame to HSV color space for better color segmentation
    cvtColor(frame, hsvImage, COLOR_BGR2HSV);

    // Define the skin color range in HSV (adjust these values as necessary)
    Scalar lowerSkinTone(0, 20, 70);
    Scalar upperSkinTone(20, 255, 255);

    // Create a mask for the skin color range
    inRange(hsvImage, lowerSkinTone, upperSkinTone, skinMask);

    // Apply morphological operations to clean up the mask
    Mat kernel = getStructuringElement(MORPH_ELLIPSE, Size(5, 5));
    erode(skinMask, skinMask, kernel);
    dilate(skinMask, skinMask, kernel);

    // Find contours in the mask
    vector<vector<Point>> contours;
    findContours(skinMask, contours, RETR_TREE, CHAIN_APPROX_SIMPLE);

    // Identify the largest contour, assuming it corresponds to the hand
    int largestContourIndex = -1;
    double maxContourArea = 0;
    for (size_t i = 0; i < contours.size(); i++) {
        double currentContourArea = contourArea(contours[i]);
        if (currentContourArea > maxContourArea) {
            maxContourArea = currentContourArea;
            largestContourIndex = i;
        }
    }

    // Proceed if a contour was found
    if (largestContourIndex != -1) {
        vector<vector<Point>> hull(1);
        vector<int> hullIndices;

        // Compute the convex hull of the largest contour
        convexHull(contours[largestContourIndex], hull[0]);
        convexHull(contours[largestContourIndex], hullIndices);

        // Draw the contour and its convex hull
        drawContours(frame, contours, largestContourIndex, Scalar(0, 255, 0), 2);
        drawContours(frame, hull, 0, Scalar(255, 0, 0), 2);

        // Find and draw convexity defects (gaps between fingers)
        vector<Vec4i> defects;
        if (hullIndices.size() > 3) {
            convexityDefects(contours[largestContourIndex], hullIndices, defects);

            for (size_t i = 0; i < defects.size(); i++) {
                int startIdx = defects[i][0];
                int endIdx = defects[i][1];
                int farIdx = defects[i][2];

                Point startPoint = contours[largestContourIndex][startIdx];
                Point endPoint = contours[largestContourIndex][endIdx];
                Point farthestPoint = contours[largestContourIndex][farIdx];

                // Draw lines between start and end points and circles at the farthest point
                line(frame, startPoint, endPoint, Scalar(0, 255, 0), 2);
                circle(frame, farthestPoint, 5, Scalar(0, 0, 255), -1);
            }
        }
    }
}

int main() {
    // Open the default camera
    VideoCapture camera(0);

    if (!camera.isOpened()) {
        cerr << "Error: Unable to access the camera." << endl;
        return -1;
    }

    while (true) {
        Mat frame;
        camera >> frame;

        if (frame.empty()) {
            cerr << "Error: Empty frame grabbed from the camera." << endl;
            break;
        }

        // Perform hand tracking on the current frame
        trackHand(frame);

        // Display the frame with the hand tracking visualization
        imshow("Hand Tracking", frame);

        // Exit if the user presses 'q'
        if (waitKey(30) == 'q') {
            break;
        }
    }

    // Release the camera and close any open windows
    camera.release();
    destroyAllWindows();
    return 0;
}
