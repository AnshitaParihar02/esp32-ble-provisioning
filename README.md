# esp32-ble-provisioning
ESP32 BLE Provisioning Example using ESP-IDF
#include <string.h>
#include <esp_system.h>
#include <esp_wifi.h>
#include <esp_ble_prov.h>

// Provisioning credentials
#define PROV_CREDENTIALS "your_prov_credentials_here"

// WiFi credentials
#define WIFI_SSID "your_wifi_ssid_here"
#define WIFI_PASSWORD "your_wifi_password_here"

void app_main(void)
{
    // Initialize WiFi
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
    ESP_ERROR_CHECK(esp_wifi_start());

    // Initialize BLE provisioning
    esp_ble_prov_config_t prov_cfg = {
        .prov_mode = ESP_BLE_PROV_MODE_BLE,
        .service_uuid = ESP_BLE_PROV_SERVICE_UUID,
        .service_instance_uuid = ESP_BLE_PROV_SERVICE_INSTANCE_UUID,
        .device_name = "ESP32 Provisioning",
        .device_service_uuid = ESP_BLE_PROV_DEVICE_SERVICE_UUID,
        .device_service_instance_uuid = ESP_BLE_PROV_DEVICE_SERVICE_INSTANCE_UUID,
        .prov_data = PROV_CREDENTIALS,
    };
    ESP_ERROR_CHECK(esp_ble_prov_init(&prov_cfg));

    // Start BLE provisioning
    ESP_ERROR_CHECK(esp_ble_prov_start());

    // Wait for provisioning to complete
    while (1) {
        esp_ble_prov_result_t result = esp_ble_prov_get_result();
        if (result == ESP_BLE_PROV_RESULT_SUCCESS) {
            break;
        }
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }

    // Connect to WiFi using provisioned credentials
    wifi_config_t wifi_cfg = {
        .sta = {
            .ssid = WIFI_SSID,
            .password = WIFI_PASSWORD,
        },
    };
    ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_cfg));
    ESP_ERROR_CHECK(esp_wifi_connect());
}
