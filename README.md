# c.github.io
using System;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Media.Imaging;
using Windows.Media.Capture;
using Windows.Media.MediaProperties;
using Windows.Storage;
using Windows.Storage.Streams;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using System.IO;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

namespace WebCamSample
{

    public sealed partial class MainPage : Page
    {
        private MediaCapture mediaCapture;
        private StorageFile photoFile;
        private readonly string PHOTO_FILE_NAME = "photo.jpg";
        private bool isPreviewing;
        private bool isRecording;
        private object info;
        private string encodeBody;
        private string filePath;

        //        public object JsonConvert { get; private set; }
        public object Formatting { get; private set; }

        #region HELPER_FUNCTIONS

        enum Action
        {
            ENABLE,
            DISABLE
        }

        private void SetInitButtonVisibility(Action action)
        {
            if (action == Action.ENABLE)
            {
                on_init.IsEnabled = true;
                // audio_init.IsEnabled = true;
            }
            else
            {
                on_init.IsEnabled = false;
                // audio_init.IsEnabled = false;
            }
        }


        private void SetVideoButtonVisibility(Action action)
        {
            if (action == Action.ENABLE)
            {
                takePhoto.IsEnabled = true;
                takePhoto.Visibility = Visibility.Visible;
            }
            else
            {
                takePhoto.IsEnabled = false;
                takePhoto.Visibility = Visibility.Collapsed;
            }
        }

        private void SetAudioButtonVisibility(Action action)
        {
            if (action == Action.ENABLE)
            { }
            else
            { }
        }
        #endregion
        public MainPage()
        {
            this.InitializeComponent();

         

            SetInitButtonVisibility(Action.ENABLE);
            SetVideoButtonVisibility(Action.DISABLE);
            SetAudioButtonVisibility(Action.DISABLE);
            isRecording = false;
            isPreviewing = false;
        }

        private async void Cleanup()
        {
            if (mediaCapture != null)
            {
                // Cleanup MediaCapture object
                if (isPreviewing)
                {
                    await mediaCapture.StopPreviewAsync();
                    captureImage.Source = null;
                    isPreviewing = false;
                }
                if (isRecording)
                {
                    await mediaCapture.StopRecordAsync();
                    isRecording = false;

                }
                mediaCapture.Dispose();
                mediaCapture = null;
            }
            SetInitButtonVisibility(Action.ENABLE);
        }

        private async void Cameraon_Click(object sender, RoutedEventArgs e)
        {
            // Disable all buttons until initialization completes

            SetInitButtonVisibility(Action.DISABLE);
            SetVideoButtonVisibility(Action.DISABLE);
            SetAudioButtonVisibility(Action.DISABLE);

            try
            {
                if (mediaCapture != null)
                {
                    // Cleanup MediaCapture object
                    if (isPreviewing)
                    {
                        await mediaCapture.StopPreviewAsync();
                        captureImage.Source = null;
                        isPreviewing = false;
                    }
                    if (isRecording)
                    {
                        await mediaCapture.StopRecordAsync();
                        isRecording = false;
                    }
                    mediaCapture.Dispose();
                    mediaCapture = null;
                }

                // Use default initialization
                mediaCapture = new MediaCapture();
                await mediaCapture.InitializeAsync();

                // Start Preview                
                previewElement.Source = mediaCapture;
                await mediaCapture.StartPreviewAsync();
                isPreviewing = true;
                status.Text = "????????????????????????????????????";

                // Enable buttons for video and photo capture
                SetVideoButtonVisibility(Action.ENABLE);

            }
            catch (Exception ex)
            {
                status.Text = "Unable to initialize camera for audio/video mode: " + ex.Message;
            }
        }

        private void cleanup_Click(object sender, RoutedEventArgs e)
        {
            SetInitButtonVisibility(Action.DISABLE);
            SetVideoButtonVisibility(Action.DISABLE);
            SetAudioButtonVisibility(Action.DISABLE);
            Cleanup();
        }
        private async void takePhoto_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                takePhoto.IsEnabled = false;
                captureImage.Source = null;

                photoFile = await KnownFolders.PicturesLibrary.CreateFileAsync(
                    PHOTO_FILE_NAME, CreationCollisionOption.GenerateUniqueName);
                ImageEncodingProperties imageProperties = ImageEncodingProperties.CreateJpeg();
                await mediaCapture.CapturePhotoToStorageFileAsync(imageProperties, photoFile);
                takePhoto.IsEnabled = true;
                status.Text = "???????????????????????????????????????" /*+ photoFile.Path*/;

                IRandomAccessStream photoStream = await photoFile.OpenReadAsync();
                BitmapImage bitmap = new BitmapImage();
                bitmap.SetSource(photoStream);
                captureImage.Source = bitmap;
            }
            catch (Exception ex)
            {
                status.Text = ex.Message;
                Cleanup();
            }
            finally
            {
                takePhoto.IsEnabled = true;

            }
        }
       

        private async void Photo_Load(object sender)
        {
            // ????????????????????????
            var info = new cameradata
            {
              data = encodeBody,
               
            };
            string json = JsonConvert.SerializeObject(info);

            var httpClient = new HttpClient();
            // ?????????????????????????????????(5???)
            httpClient.Timeout = TimeSpan.FromMilliseconds(5000);
            var content = new StringContent(json, Encoding.UTF8, "application/json");


            try
            {
                /*// Basic??????????????????????????????????????????
                var userName = "user";
                var userPassword = "676a0f33-d85e-46e1-b9ad-2aa62c36a156";
*/
                // ????????????????????????
                var request = new HttpRequestMessage
                {
                    Method = HttpMethod.Post,
                    RequestUri = new Uri("http://192.168.10.51:10000/test1/index")
                };

               /* // Basic??????????????????????????????
                request.Headers.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue(
                    "Basic",
                    Convert.ToBase64String(Encoding.ASCII.GetBytes(string.Format("{0}:{1}", userName, userPassword))));*/

                var response = await httpClient.PostAsync("http://192.168.10.51:10000/test1/index", content);
                var resString = await response.Content.ReadAsStringAsync();

                JObject jsonObj = JObject.Parse(resString);
               // Console.WriteLine("Key1={0}", jsonObj["key1"]);

                status.Text = "???????????????" + jsonObj["msg"];
     
            }
            catch (Exception ex)
            {
                status.Text = "?????????";
            }

        }
        // Base64?????????????????????????????????????????????
        public class Base64
        {
            // ?????????????????????????????????UTF-8?????????Base64????????????????????????
            public static string Encode(string str)
            {
                return Encode(str, Encoding.UTF8);
            }
            // ?????????????????????????????????????????????????????????
            public static string Encode(string str, Encoding encode)
            {
                return Convert.ToBase64String(encode.GetBytes(str));
            }

            // ????????????Base64????????????UTF-8??????????????????????????????????????????
            public static string Decode(string base64Str)
            {
                return Decode(base64Str, Encoding.UTF8);
            }
            // ?????????????????????????????????????????????????????????
            public static string Decode(string base64Str, Encoding encode)
            {
                return encode.GetString(Convert.FromBase64String(base64Str));
            }

            public string ReadWithEncode(string filePath)
            {
                Task<string> task = Task.Run(() => {
                    return Convert.ToBase64String(File.ReadAllBytes(filePath));
                });
                task.Wait();

                return task.Result;
               // return Convert.ToBase64String(File.ReadAllBytes(filePath));
            }

            // Base64???????????????????????????????????????????????????????????????????????????
            public static void SaveWithDecode(string base64Str, string savePath)
            {
                byte[] barray = Convert.FromBase64String(base64Str);
                using (var fs = new FileStream(savePath, FileMode.Create))
                {
                    fs.Write(barray, 0, barray.Length);
                }
            }
        }
       

        private void trans_Click(object sender, RoutedEventArgs e)
        {
            MainPage.Base64 base64 = new MainPage.Base64();
            // ??????????????????????????????
            filePath = photoFile.Path;

            // ??????????????????????????????Base64??????????????????
            encodeBody = base64.ReadWithEncode(filePath);

            try
            {
                transmission.IsEnabled = false;
                captureImage.Source = null;

                transmission.IsEnabled = true;
                Photo_Load(info);
            }
            catch (Exception ex)
            {
                status.Text = ex.Message;
               
                Cleanup();
            }
            finally
            {
                transmission.IsEnabled = true;
            }
        }
    }
    public class cameradata
    {
        [JsonProperty]
        public string data { get; set; }
       
    }
}
