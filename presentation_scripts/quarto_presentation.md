---
title: "Flight Support: Azure API for Message Generation"
author: "Your Name"
format: 
  revealjs:
    theme: simple
    transition: slide
    slide-number: true
    code-fold: false
    highlight-style: github
---

## Introduction

`flight_support.R` provides an aviation-themed interface to the Azure API for generating message variations.

Key features:

- Seamless dependency management
- API credential handling
- Flexible message generation with system and user roles
- Multiple response variations with temperature control

## Core Functions

The package uses an aviation theme throughout:

- `clear_runway()`: Sets up dependencies
- `airport_security()`: Manages API credentials
- `set_course()`: Sends individual API requests
- `pilot_message()`: Main function for generating message variations

## Function Architecture

```r
pilot_message(
  base_prompt,       # User message content
  system_prompt,     # System message to guide behavior
  additional_information, # Optional content to append
  connections = 4,   # Number of variations to generate
  temperature = 0,   # Controls randomness (0-2)
  itinerary = FALSE  # Return format control
)
```

## Under the Hood: Dependency Management

```r
clear_runway <- function() {
  required_packages <- c("httr2", "tidyverse")
  
  for(pkg in required_packages) {
    if(!requireNamespace(pkg, quietly = TRUE)) {
      install.packages(pkg)
    }
    library(pkg, character.only = TRUE)
  }
}
```

## Under the Hood: Credential Management

```r
airport_security <- function() {
  if(!exists("flight_credentials", envir = .GlobalEnv)) {
    # Check for API file or prompt for credentials
    if(file.exists("azure.api")) {
      api <- readr::read_csv("azure.api", show_col_types = FALSE)      
      api_key <- api$key[[1]] 
      endpoint <- api$endpoint[[1]] 
    } else {
      api_key <- readline("Enter your API key: ")
      endpoint <- readline("Enter your API endpoint: ")
    }
    
    # Store in global environment
    flight_credentials <- list(api_key = api_key, endpoint = endpoint)
    assign("flight_credentials", flight_credentials, envir = .GlobalEnv)
  }
  
  return(get("flight_credentials", envir = .GlobalEnv))
}
```

## Under the Hood: API Request Formation

```r
set_course <- function(prompt, system_content = NULL, temperature = 0) {
  credentials <- airport_security()
  
  # Construct messages array
  messages <- list()
  if (!is.null(system_content)) {
    messages[[length(messages) + 1]] <- list(
      role = "system", 
      content = system_content
    )
  }
  messages[[length(messages) + 1]] <- list(
    role = "user", 
    content = prompt
  )
  
  # Build and send request
  request <- httr2::request(credentials$endpoint) |>
    httr2::req_headers(
      Authorization = paste("Bearer", credentials$api_key),
      "Content-Type" = "application/json"
    ) |>
    httr2::req_body_json(list(
      model = "optimize-v2",
      messages = messages,
      max_tokens = 600,
      temperature = temperature
    ))
  
  response <- httr2::req_perform(request)
  httr2::resp_body_json(response)
}
```

## The Main Function: pilot_message()

```r
pilot_message <- function(base_prompt, 
                          system_prompt = NULL,
                          additional_information = NULL,
                          connections = 4,
                          temperature = 0,
                          itinerary = FALSE) {
  
  # Enhance prompt if additional info provided
  enhanced_prompt <- base_prompt
  if(!is.null(additional_information)) {
    enhanced_prompt <- paste0(enhanced_prompt, 
                            "\nAdditional information: ", 
                            additional_information)
  }
  
  # Generate multiple variations
  all_messages <- list()
  for(i in 1:connections) {
    response <- set_course(enhanced_prompt, 
                           system_content = system_prompt, 
                           temperature = temperature)
    
    if (!is.null(response) && !is.null(response$choices) && 
        length(response$choices) > 0) {
      raw_result <- response$choices[[1]]$message$content
      all_messages[[i]] <- trimws(raw_result)
    } else {
      all_messages[[i]] <- paste("API response error for connection", i)
    }
  }
  
  all_messages <- unlist(all_messages)
  
  # Return format control
  if (itinerary) {
    return(list(
      system = system_prompt,
      prompt = enhanced_prompt,
      response = all_messages
    ))
  } else {
    return(all_messages)
  }
}
```

## Example Usage

```r
# Define prompts
system_prompt <- "You are a helpful assistant."
user_prompt <- "Suggest three ways to improve customer satisfaction."

# Generate 3 variations with moderate creativity
responses <- pilot_message(
  system_prompt = system_prompt,
  base_prompt = user_prompt,
  connections = 3,
  temperature = 0.7
)

# Display results
cat(responses[1])
```

## Key Benefits

- **Flexibility**: Control system messages, user messages, and response variations
- **Persistence**: Credentials stored in session for reuse
- **Convenience**: Automatic package installation
- **Customization**: Temperature parameter for creativity control
- **Organization**: Optional structured return format with 'itinerary'

## Questions?

Thank you!

---

Contact information:
- Email: your.email@example.com
- GitHub: github.com/yourusername
