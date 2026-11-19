[script_selenium_lab_activity.txt..py](https://github.com/user-attachments/files/35094920/script_selenium_lab_activity.txt.py)
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import Select, WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


def validate_form(text, password, dropdown_value, radio_selected):
    errors = []

    if not text.strip():
        errors.append("Text box is empty.")

    if not password:
        errors.append("Password field is empty.")
    elif len(password) < 8:
        errors.append("Password must contain at least 8 characters.")

    if dropdown_value.strip() == "" or dropdown_value == "Open this select menu":
        errors.append("Please choose an option from the dropdown.")

    if not radio_selected:
        errors.append("Please select a radio button.")

    return errors


driver = webdriver.Chrome()

try:
    driver.get("https://www.selenium.dev/selenium/web/web-form.html")
    wait = WebDriverWait(driver, 10)

    print("Page title:", driver.title)

    # Locate the five required form elements.
    text_box = wait.until(
        EC.presence_of_element_located((By.NAME, "my-text"))
    )
    password_field = driver.find_element(By.NAME, "my-password")
    dropdown_element = driver.find_element(By.NAME, "my-select")
    dropdown = Select(dropdown_element)
    radio_buttons = driver.find_elements(
        By.CSS_SELECTOR, "input[type='radio'][name='my-radio']"
    )
    submit_button = driver.find_element(By.CSS_SELECTOR, "button")

    # TEST 1: EMPTY FIELDS
    print("\nTEST 1: Empty Fields")

    text_box.clear()
    password_field.clear()
    dropdown.select_by_index(0)

    # Clear any preselected radio button for the empty-field test.
    for radio in radio_buttons:
        driver.execute_script("arguments[0].checked = false;", radio)

    selected_option = dropdown.first_selected_option
    empty_errors = validate_form(
        text_box.get_attribute("value"),
        password_field.get_attribute("value"),
        selected_option.text.strip(),
        any(radio.is_selected() for radio in radio_buttons)
    )

    if empty_errors:
        print("PASS: Empty fields were detected.")
        for error in empty_errors:
            print("-", error)
    else:
        print("FAIL: Empty fields were not detected.")

    # TEST 2: INVALID INPUT
    print("\nTEST 2: Invalid Input")

    text_box.clear()
    text_box.send_keys("Student")
    password_field.clear()
    password_field.send_keys("123")

    dropdown.select_by_visible_text("One")
    radio_buttons[-1].click()

    selected_option = dropdown.first_selected_option
    invalid_errors = validate_form(
        text_box.get_attribute("value"),
        password_field.get_attribute("value"),
        selected_option.text.strip(),
        any(radio.is_selected() for radio in radio_buttons)
    )

    if invalid_errors:
        print("PASS: Invalid input was detected.")
        for error in invalid_errors:
            print("-", error)
    else:
        print("FAIL: Invalid input was not detected.")

    # TEST 3: SUCCESSFUL FORM SUBMISSION
    print("\nTEST 3: Successful Form Submission")

    text_box.clear()
    text_box.send_keys("Selenium Student")

    password_field.clear()
    password_field.send_keys("Selenium123")

    dropdown.select_by_visible_text("Two")
    radio_buttons[-1].click()

    selected_option = dropdown.first_selected_option
    final_errors = validate_form(
        text_box.get_attribute("value"),
        password_field.get_attribute("value"),
        selected_option.text.strip(),
        any(radio.is_selected() for radio in radio_buttons)
    )

    if not final_errors:
        submit_button.click()

        message = wait.until(
            EC.visibility_of_element_located((By.ID, "message"))
        )

        print("Submission message:", message.text)
        assert message.text == "Received!"
        print("PASS: Form submitted successfully!")
    else:
        print("FAIL: Form still has errors.")
        for error in final_errors:
            print("-", error)

finally:
    driver.quit()
    print("\nBrowser closed.")
