# Fit-IA
"use client";
import React from "react";

import { useUpload } from "../utilities/runtime-helpers";

function MainComponent() {
  const [activeTab, setActiveTab] = useState("home");
  const [userPoints, setUserPoints] = useState(0);
  const [userLeague, setUserLeague] = useState("Bronce 2");
  const [userAvatar, setUserAvatar] = useState("/default-avatar.png");
  const [userDescription, setUserDescription] = useState("");
  const [userSocialLinks, setUserSocialLinks] = useState([]);
  const [isEditingProfile, setIsEditingProfile] = useState(false);
  const [selectedProfileAvatar, setSelectedProfileAvatar] = useState(null);
  const [workoutPlan, setWorkoutPlan] = useState({
    Lunes: "",
    Martes: "",
    Miércoles: "",
    Jueves: "",
    Viernes: "",
    Sábado: "",
    Domingo: "",
  });
  const [dietPlan, setDietPlan] = useState({
    Lunes: "",
    Martes: "",
    Miércoles: "",
    Jueves: "",
    Viernes: "",
    Sábado: "",
    Domingo: "",
  });
  const [userInput, setUserInput] = useState("");
  const [followers, setFollowers] = useState([]);
  const [following, setFollowing] = useState([]);
  const [searchQuery, setSearchQuery] = useState("");
  const [searchResults, setSearchResults] = useState([]);
  const [selectedDay, setSelectedDay] = useState("Lunes");
  const [showChat, setShowChat] = useState(false);
  const [selectedUser, setSelectedUser] = useState(null);
  const [selectedSticker, setSelectedSticker] = useState(null);
  const [showStickers, setShowStickers] = useState(false);
  const [upload, { loading }] = useUpload();
  const [dailyProgress, setDailyProgress] = useState({
    workout: {},
    diet: {},
  });
  const [notificationsEnabled, setNotificationsEnabled] = useState(false);
  const [installPrompt, setInstallPrompt] = useState(null);
  const [avatares, setAvatares] = useState([]);
  const days = [
    "Lunes",
    "Martes",
    "Miércoles",
    "Jueves",
    "Viernes",
    "Sábado",
    "Domingo",
  ];
  const [posts, setPosts] = useState([
    {
      id: 1,
      user: "FitUser1",
      image: "/workout-1.jpg",
      likes: 45,
      description: "¡Otro día más superándome! 💪",
      avatar: "/avatar-1.png",
      music: "Canción motivadora.mp3",
      avatarOverlay: "/avatars/arnold-3d-full.png",
    },
  ]);
  const handleMediaUpload = async (e) => {
    const file = e.target.files[0];
    const { url, error } = await upload({ file });
    if (error) {
      console.error(error);
      return;
    }
    setSelectedProfileAvatar(url);
  };
  const handleSavePlan = (type, plan) => {
    if (type === "workout") {
      setWorkoutPlan({
        ...workoutPlan,
        [selectedDay]: plan,
      });
      scheduleNotification(selectedDay);
    } else {
      setDietPlan({
        ...dietPlan,
        [selectedDay]: plan,
      });
    }
  };
  const handleCheckProgress = (type, day, completed) => {
    setDailyProgress((prev) => ({
      ...prev,
      [type]: {
        ...prev[type],
        [day]: completed,
      },
    }));
  };
  const handleSendSticker = (stickerUrl) => {
    setSelectedSticker(stickerUrl);
    setShowStickers(false);
  };
  const scheduleNotification = (day) => {
    if (!notificationsEnabled) return;
    const now = new Date();
    const scheduledTime = new Date();
    scheduledTime.setHours(8, 0, 0, 0);
    if (scheduledTime < now) {
      scheduledTime.setDate(scheduledTime.getDate() + 1);
    }
    const timeUntilNotification = scheduledTime - now;
    setTimeout(() => {
      new Notification("¡Hora de entrenar!", {
        body: `¡Es hora de tu entrenamiento de ${day}!`,
        icon: "/logo.png",
      });
    }, timeUntilNotification);
  };
  const handleSaveProfile = () => {
    if (selectedProfileAvatar) {
      setUserAvatar(selectedProfileAvatar);
    }
    setIsEditingProfile(false);
  };
  const handleAddSocialLink = (link) => {
    setUserSocialLinks([...userSocialLinks, link]);
  };

  useEffect(() => {
    document.title = "FitPoints - Tu entrenador personal";
    const meta = document.createElement("meta");
    meta.name = "description";
    meta.content =
      "Alcanza tus objetivos fitness con FitPoints, la app que gamifica tu entrenamiento con avatares 3D y planes personalizados";
    document.head.appendChild(meta);

    const link = document.createElement("link");
    link.rel = "manifest";
    link.href = "/manifest.json";
    document.head.appendChild(link);

    if ("serviceWorker" in navigator) {
      window.addEventListener("load", () => {
        navigator.serviceWorker
          .register("/sw.js")
          .then((registration) => {
            console.log("SW registrado:", registration);
          })
          .catch((error) => {
            console.log("SW error:", error);
          });
      });
    }

    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault();
      setInstallPrompt(e);
    });

    if ("Notification" in window) {
      Notification.requestPermission();
    }

    const generateAvatarWithAI = async (prompt) => {
      try {
        const response = await fetch("/api/stable-diffusion", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            prompt: `Generate a highly detailed 3D fitness character avatar in a dynamic pose with professional lighting, wearing modern fitness attire. The character should have a muscular build and be rendered in a photorealistic style with vibrant colors. ${prompt}`,
            negative_prompt:
              "deformed, unrealistic, low quality, blurry, bad anatomy",
            steps: 30,
            width: 512,
            height: 512,
            guidance_scale: 7.5,
          }),
        });

        const data = await response.json();
        return data.imageUrl;
      } catch (error) {
        console.error("Error generando avatar:", error);
        return null;
      }
    };
    const generateInitialAvatars = async () => {
      if (avatares.length < 50) {
        const promesas = [];
        const avataresFaltantes = 50 - avatares.length;

        for (let i = 0; i < avataresFaltantes; i++) {
          promesas.push(
            generateAvatarWithAI(
              `Unique fitness character ${
                i + 1
              } of 50, distinct personality and style`
            )
          );
        }

        try {
          const imagenesGeneradas = await Promise.all(promesas);
          const nuevosAvatares = imagenesGeneradas.map((imagen, index) => ({
            id: avatares.length + index + 1,
            nombre: `Héroe Fitness ${avatares.length + index + 1}`,
            imagen: imagen,
            imagenBebe: imagen,
            objeto: {
              nombre: `Equipo Especial ${avatares.length + index + 1}`,
              imagen: `/objects/special-${avatares.length + index + 1}.png`,
              animacion: "flex",
            },
            stickers: [
              `/stickers/hero-${avatares.length + index + 1}-1.png`,
              `/stickers/hero-${avatares.length + index + 1}-2.png`,
              `/stickers/hero-${avatares.length + index + 1}-3.png`,
            ],
            desbloqueado: false,
          }));

          setAvatares((prev) => [...prev, ...nuevosAvatares]);
        } catch (error) {
          console.error("Error generando avatares iniciales:", error);
        }
      }
    };
    const checkForNewMonthlyAvatar = async () => {
      const lastUpdate = localStorage.getItem("lastAvatarUpdate");
      const now = new Date();

      if (!lastUpdate || new Date(lastUpdate).getMonth() !== now.getMonth()) {
        const newAvatarImage = await generateAvatarWithAI(
          "Monthly special fitness character with unique and distinctive features"
        );

        if (newAvatarImage) {
          const newAvatar = {
            id: avatares.length + 1,
            nombre: `Héroe Fitness del Mes ${now.toLocaleString("es-ES", {
              month: "long",
              year: "numeric",
            })}`,
            imagen: newAvatarImage,
            imagenBebe: newAvatarImage,
            objeto: {
              nombre: `Equipo Especial Mensual`,
              imagen: `/objects/special-monthly-${now.getMonth()}.png`,
              animacion: "flex",
            },
            stickers: [
              `/stickers/monthly-${now.getMonth()}-1.png`,
              `/stickers/monthly-${now.getMonth()}-2.png`,
              `/stickers/monthly-${now.getMonth()}-3.png`,
            ],
            desbloqueado: false,
          };

          setAvatares((prev) => [...prev, newAvatar]);
          localStorage.setItem("lastAvatarUpdate", now.toISOString());
        }
      }
    };

    generateInitialAvatars();
    checkForNewMonthlyAvatar();
    const monthlyCheck = setInterval(checkForNewMonthlyAvatar, 86400000);

    const checkScheduledWorkouts = () => {
      const now = new Date();
      const day = days[now.getDay()];
      if (workoutPlan[day] && !dailyProgress.workout[day]) {
        new Notification("¡Hora de entrenar!", {
          body: "¡No olvides tu entrenamiento de hoy!",
          icon: "/logo.png",
        });
      }
    };
    const interval = setInterval(checkScheduledWorkouts, 3600000);
    return () => {
      clearInterval(interval);
      clearInterval(monthlyCheck);
    };
  }, []);

  return (
    <div className="flex flex-col h-screen bg-gray-100">
      <div className="flex-1 overflow-y-auto">
        {activeTab === "home" && (
          <div className="p-4 space-y-4">
            <div className="bg-white rounded-lg p-4 shadow">
              <div className="flex items-center space-x-4">
                <img
                  src={userAvatar}
                  alt="Avatar de usuario"
                  className="w-16 h-16 rounded-full"
                />
                <div>
                  <div className="font-bold font-roboto">{userLeague}</div>
                  <div className="text-sm text-gray-500">
                    FitPoints: {userPoints}
                  </div>
                </div>
              </div>
            </div>
            <div className="bg-white rounded-lg p-4 shadow">
              <h2 className="font-bold font-roboto mb-4">Tu Progreso Diario</h2>
              <div className="flex justify-between">
                <div className="text-center">
                  <div className="text-2xl font-bold text-green-500">5/5</div>
                  <div className="text-sm">Ejercicios</div>
                </div>
                <div className="text-center">
                  <div className="text-2xl font-bold text-blue-500">3/5</div>
                  <div className="text-sm">Dieta</div>
                </div>
              </div>
            </div>
            {posts.map((post) => (
              <div key={post.id} className="bg-white rounded-lg shadow">
                <div className="p-4 flex items-center justify-between">
                  <div className="flex items-center">
                    <img
                      src={post.avatar}
                      alt="Avatar"
                      className="w-8 h-8 rounded-full"
                    />
                    <span className="ml-2 font-roboto">{post.user}</span>
                  </div>
                  <button
                    onClick={() =>
                      setSelectedUser({ name: post.user, avatar: post.avatar })
                    }
                  >
                    <i className="fas fa-comment-dots"></i>
                  </button>
                </div>
                <div className="relative">
                  {post.avatarOverlay && (
                    <img
                      src={post.avatarOverlay}
                      alt="Avatar overlay"
                      className="absolute top-2 right-2 w-16 h-16 object-contain"
                    />
                  )}
                  {post.video ? (
                    <video
                      src={post.image}
                      className="w-full h-[300px] object-cover"
                      controls
                    />
                  ) : (
                    <img
                      src={post.image}
                      alt="Post de ejercicio"
                      className="w-full h-[300px] object-cover"
                    />
                  )}
                  {post.music && (
                    <div className="absolute bottom-2 left-2 bg-black bg-opacity-50 text-white px-2 py-1 rounded">
                      <i className="fas fa-music mr-2"></i>
                      {post.music}
                    </div>
                  )}
                </div>
                <div className="p-4">
                  <div className="flex items-center space-x-4">
                    <button className="text-2xl">
                      <i className="far fa-heart"></i>
                    </button>
                    <button className="text-2xl">
                      <i className="far fa-comment"></i>
                    </button>
                    <button className="text-2xl">
                      <i className="fas fa-share"></i>
                    </button>
                  </div>
                  <div className="mt-2">{post.likes} likes</div>
                  <div className="mt-1">{post.description}</div>
                </div>
              </div>
            ))}
          </div>
        )}

        {activeTab === "workout" && (
          <div className="p-4 space-y-4">
            <div className="bg-white rounded-lg p-4 shadow">
              <div className="flex space-x-2 mb-4 overflow-x-auto">
                {days.map((day) => (
                  <button
                    key={day}
                    onClick={() => setSelectedDay(day)}
                    className={`px-4 py-2 rounded ${
                      selectedDay === day
                        ? "bg-blue-500 text-white"
                        : "bg-gray-200"
                    }`}
                  >
                    {day}
                  </button>
                ))}
              </div>
              <div className="p-4 space-y-4">
                <div className="bg-white rounded-lg p-4 shadow">
                  <h2 className="font-bold font-roboto text-xl mb-4">
                    Tu Plan de Ejercicios
                  </h2>
                  <textarea
                    className="w-full p-2 border rounded mb-4"
                    placeholder="Describe tus objetivos, experiencia y preferencias de ejercicio..."
                    value={userInput}
                    onChange={(e) => setUserInput(e.target.value)}
                  />
                  <button className="bg-blue-500 text-white px-4 py-2 rounded">
                    Generar Plan Personalizado
                  </button>
                  {workoutPlan[selectedDay] && (
                    <div className="mt-4 p-4 bg-gray-50 rounded">
                      <div className="whitespace-pre-wrap">
                        {workoutPlan[selectedDay]}
                      </div>
                      <div className="mt-4 flex items-center">
                        <input
                          type="checkbox"
                          checked={dailyProgress.workout[selectedDay] || false}
                          onChange={(e) =>
                            handleCheckProgress(
                              "workout",
                              selectedDay,
                              e.target.checked
                            )
                          }
                          className="mr-2"
                        />
                        <span>Marcar como completado</span>
                      </div>
                    </div>
                  )}
                </div>
              </div>
            </div>
          </div>
        )}

        {activeTab === "diet" && (
          <div className="p-4 space-y-4">
            <div className="bg-white rounded-lg p-4 shadow">
              <div className="flex space-x-2 mb-4 overflow-x-auto">
                {days.map((day) => (
                  <button
                    key={day}
                    onClick={() => setSelectedDay(day)}
                    className={`px-4 py-2 rounded ${
                      selectedDay === day
                        ? "bg-green-500 text-white"
                        : "bg-gray-200"
                    }`}
                  >
                    {day}
                  </button>
                ))}
              </div>
              <div className="p-4 space-y-4">
                <div className="bg-white rounded-lg p-4 shadow">
                  <h2 className="font-bold font-roboto text-xl mb-4">
                    Tu Plan de Alimentación
                  </h2>
                  <textarea
                    className="w-full p-2 border rounded mb-4"
                    placeholder="Describe tus objetivos, alergias y preferencias alimenticias..."
                    value={userInput}
                    onChange={(e) => setUserInput(e.target.value)}
                  />
                  <button className="bg-green-500 text-white px-4 py-2 rounded">
                    Generar Plan Nutricional
                  </button>
                  {dietPlan[selectedDay] && (
                    <div className="mt-4 p-4 bg-gray-50 rounded">
                      <div className="whitespace-pre-wrap">
                        {dietPlan[selectedDay]}
                      </div>
                      <div className="mt-4 flex items-center">
                        <input
                          type="checkbox"
                          checked={dailyProgress.diet[selectedDay] || false}
                          onChange={(e) =>
                            handleCheckProgress(
                              "diet",
                              selectedDay,
                              e.target.checked
                            )
                          }
                          className="mr-2"
                        />
                        <span>Marcar como completado</span>
                      </div>
                    </div>
                  )}
                </div>
              </div>
            </div>
          </div>
        )}

        {activeTab === "search" && (
          <div className="p-4 space-y-4">
            <div className="bg-white rounded-lg p-4 shadow">
              <div className="flex space-x-2">
                <input
                  type="text"
                  placeholder="Buscar ejercicios o recetas..."
                  className="flex-1 p-2 border rounded"
                  value={searchQuery}
                  onChange={(e) => setSearchQuery(e.target.value)}
                />
                <button className="bg-blue-500 text-white px-4 py-2 rounded">
                  <i className="fas fa-search"></i>
                </button>
              </div>
              <div className="grid grid-cols-2 gap-4 mt-4">
                {searchResults.map((result, index) => (
                  <div key={index} className="rounded overflow-hidden shadow">
                    {result.type === "video" ? (
                      <video
                        src={result.url}
                        controls
                        className="w-full h-48 object-cover"
                      />
                    ) : (
                      <img
                        src={result.url}
                        alt={result.title}
                        className="w-full h-48 object-cover"
                      />
                    )}
                    <div className="p-2">
                      <h3 className="font-bold text-sm">{result.title}</h3>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {activeTab === "profile" && (
          <div className="p-4 space-y-4">
            <div className="bg-white rounded-lg p-4 shadow">
              <div className="flex justify-between items-center mb-6">
                <h2 className="font-bold font-roboto text-xl">Tu Perfil</h2>
                <button
                  onClick={() => setIsEditingProfile(!isEditingProfile)}
                  className="text-blue-500 hover:text-blue-600"
                >
                  <i
                    className={`fas ${
                      isEditingProfile ? "fa-times" : "fa-edit"
                    }`}
                  ></i>
                </button>
              </div>

              {isEditingProfile ? (
                <div className="space-y-4">
                  <div className="flex items-center space-x-4">
                    <img
                      src={selectedProfileAvatar || userAvatar}
                      alt="Avatar de usuario"
                      className="w-24 h-24 rounded-full object-cover"
                    />
                    <div className="space-y-2">
                      <input
                        type="file"
                        accept="image/*"
                        onChange={handleMediaUpload}
                        className="hidden"
                        id="avatar-upload"
                      />
                      <label
                        htmlFor="avatar-upload"
                        className="bg-blue-500 text-white px-4 py-2 rounded cursor-pointer hover:bg-blue-600"
                      >
                        Subir Foto
                      </label>
                      <button
                        onClick={() => setSelectedProfileAvatar(null)}
                        className="block text-red-500 hover:text-red-600"
                      >
                        Eliminar Foto
                      </button>
                    </div>
                  </div>

                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-2">
                      Descripción
                    </label>
                    <textarea
                      value={userDescription}
                      onChange={(e) => setUserDescription(e.target.value)}
                      placeholder="Cuéntanos sobre ti... (opcional)"
                      className="w-full p-2 border rounded min-h-[100px]"
                    />
                  </div>

                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-2">
                      Redes Sociales
                    </label>
                    <div className="space-y-2">
                      {userSocialLinks.map((link, index) => (
                        <div
                          key={index}
                          className="flex items-center space-x-2"
                        >
                          <input
                            type="text"
                            value={link}
                            onChange={(e) => {
                              const newLinks = [...userSocialLinks];
                              newLinks[index] = e.target.value;
                              setUserSocialLinks(newLinks);
                            }}
                            className="flex-1 p-2 border rounded"
                            placeholder="https://"
                          />
                          <button
                            onClick={() => {
                              const newLinks = userSocialLinks.filter(
                                (_, i) => i !== index
                              );
                              setUserSocialLinks(newLinks);
                            }}
                            className="text-red-500 hover:text-red-600"
                          >
                            <i className="fas fa-trash"></i>
                          </button>
                        </div>
                      ))}
                      <button
                        onClick={() => handleAddSocialLink("")}
                        className="text-blue-500 hover:text-blue-600"
                      >
                        <i className="fas fa-plus mr-2"></i>
                        Añadir Red Social
                      </button>
                    </div>
                  </div>

                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-2">
                      Usar Avatar como Foto de Perfil
                    </label>
                    <div className="grid grid-cols-4 gap-2">
                      {avatares
                        .filter((a) => a.desbloqueado)
                        .map((avatar) => (
                          <button
                            key={avatar.id}
                            onClick={() =>
                              setSelectedProfileAvatar(avatar.imagen)
                            }
                            className={`relative rounded overflow-hidden ${
                              selectedProfileAvatar === avatar.imagen
                                ? "ring-2 ring-blue-500"
                                : ""
                            }`}
                          >
                            <img
                              src={avatar.imagen}
                              alt={avatar.nombre}
                              className="w-full h-16 object-cover"
                            />
                          </button>
                        ))}
                    </div>
                  </div>

                  <div className="flex justify-end space-x-2">
                    <button
                      onClick={() => setIsEditingProfile(false)}
                      className="px-4 py-2 border rounded text-gray-600 hover:bg-gray-50"
                    >
                      Cancelar
                    </button>
                    <button
                      onClick={handleSaveProfile}
                      className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
                    >
                      Guardar Cambios
                    </button>
                  </div>
                </div>
              ) : (
                <div className="space-y-4">
                  <div className="flex items-center space-x-4">
                    <img
                      src={userAvatar}
                      alt="Avatar de usuario"
                      className="w-24 h-24 rounded-full object-cover"
                    />
                    <div>
                      <div className="font-bold text-xl">{userLeague}</div>
                      <div className="text-gray-500">
                        FitPoints: {userPoints}
                      </div>
                    </div>
                  </div>

                  {userDescription && (
                    <div className="mt-4">
                      <p className="whitespace-pre-wrap">{userDescription}</p>
                    </div>
                  )}

                  {userSocialLinks.length > 0 && (
                    <div className="mt-4">
                      <h3 className="font-medium mb-2">Redes Sociales</h3>
                      <div className="flex flex-wrap gap-2">
                        {userSocialLinks.map((link, index) => (
                          <a
                            key={index}
                            href={link}
                            target="_blank"
                            rel="noopener noreferrer"
                            className="text-blue-500 hover:text-blue-600"
                          >
                            <i className="fas fa-link mr-1"></i>
                            {new URL(link).hostname}
                          </a>
                        ))}
                      </div>
                    </div>
                  )}
                </div>
              )}
            </div>

            <div className="bg-white rounded-lg p-4 shadow">
              <h2 className="font-bold font-roboto text-xl mb-4">
                Avatares Coleccionables
              </h2>
              <div className="grid grid-cols-2 md:grid-cols-3 gap-4">
                {avatares.map((avatar) => (
                  <div key={avatar.id} className="relative">
                    <img
                      src={avatar.imagen}
                      alt={avatar.nombre}
                      className={`w-full h-32 object-cover rounded ${
                        !avatar.desbloqueado && "filter grayscale"
                      }`}
                    />
                    <div className="absolute bottom-0 left-0 right-0 bg-black bg-opacity-50 text-white p-2 text-sm">
                      {avatar.nombre}
                      {!avatar.desbloqueado && (
                        <i className="fas fa-lock ml-2"></i>
                      )}
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {activeTab === "legal" && (
          <div className="p-4 space-y-4">
            <div className="bg-white rounded-lg p-4 shadow">
              <h2 className="font-bold font-roboto text-xl mb-4">
                Política de Privacidad
              </h2>
              <div className="prose">
                <p>Última actualización: 2025</p>
                <h3>1. Información que recopilamos</h3>
                <p>
                  Recopilamos información necesaria para mejorar tu experiencia
                  de entrenamiento, incluyendo:
                </p>
                <ul>
                  <li>Información de perfil</li>
                  <li>Datos de entrenamiento</li>
                  <li>Preferencias de ejercicios</li>
                  <li>Progreso físico</li>
                </ul>

                <h3>2. Uso de la información</h3>
                <p>Utilizamos tu información para:</p>
                <ul>
                  <li>Personalizar tu experiencia</li>
                  <li>Mejorar nuestros servicios</li>
                  <li>Enviar notificaciones relevantes</li>
                </ul>

                <h3>3. Compartir información</h3>
                <p>
                  No compartimos tu información personal con terceros sin tu
                  consentimiento.
                </p>
              </div>

              <h2 className="font-bold font-roboto text-xl mt-8 mb-4">
                Términos de Uso
              </h2>
              <div className="prose">
                <p>Al usar FitPoints, aceptas:</p>
                <ul>
                  <li>Usar la aplicación de manera responsable</li>
                  <li>No compartir contenido inapropiado</li>
                  <li>Respetar a otros usuarios</li>
                  <li>Seguir las pautas de la comunidad</li>
                </ul>

                <h3>Descargo de responsabilidad</h3>
                <p>
                  Consulta con un profesional de la salud antes de comenzar
                  cualquier programa de ejercicios.
                </p>
              </div>
            </div>
          </div>
        )}
      </div>

      <div className="bg-white border-t flex justify-around p-4">
        <button
          onClick={() => setActiveTab("home")}
          className={`text-2xl ${
            activeTab === "home" ? "text-blue-500" : "text-gray-500"
          }`}
        >
          <i className="fas fa-home"></i>
        </button>
        <button
          onClick={() => setActiveTab("workout")}
          className={`text-2xl ${
            activeTab === "workout" ? "text-blue-500" : "text-gray-500"
          }`}
        >
          <i className="fas fa-dumbbell"></i>
        </button>
        <button
          onClick={() => setActiveTab("diet")}
          className={`text-2xl ${
            activeTab === "diet" ? "text-blue-500" : "text-gray-500"
          }`}
        >
          <i className="fas fa-utensils"></i>
        </button>
        <button
          onClick={() => setActiveTab("search")}
          className={`text-2xl ${
            activeTab === "search" ? "text-blue-500" : "text-gray-500"
          }`}
        >
          <i className="fas fa-search"></i>
        </button>
        <button
          onClick={() => setActiveTab("profile")}
          className={`text-2xl ${
            activeTab === "profile" ? "text-blue-500" : "text-gray-500"
          }`}
        >
          <i className="fas fa-user"></i>
        </button>
        <button
          onClick={() => setActiveTab("legal")}
          className={`text-2xl ${
            activeTab === "legal" ? "text-blue-500" : "text-gray-500"
          }`}
        >
          <i className="fas fa-file-contract"></i>
        </button>
      </div>

      {showChat && selectedUser && (
        <div className="fixed bottom-20 right-4 w-80 h-96 bg-white rounded-lg shadow-lg">
          <div className="p-4 border-b flex justify-between items-center">
            <div className="flex items-center space-x-2">
              <img
                src={selectedUser.avatar}
                alt="Chat user"
                className="w-8 h-8 rounded-full"
              />
              <span>{selectedUser.name}</span>
            </div>
            <button onClick={() => setShowChat(false)}>
              <i className="fas fa-times"></i>
            </button>
          </div>
          <div className="h-72 overflow-y-auto p-4">
            {selectedSticker && (
              <div className="flex justify-end mb-4">
                <img
                  src={selectedSticker}
                  alt="Sticker enviado"
                  className="w-24 h-24 object-contain"
                />
              </div>
            )}
          </div>
          <div className="p-4 border-t">
            <div className="flex items-center space-x-2">
              <input
                type="text"
                placeholder="Escribe un mensaje..."
                className="flex-1 p-2 border rounded"
              />
              <button
                onClick={() => setShowStickers(!showStickers)}
                className="text-xl text-gray-500"
              >
                <i className="far fa-smile"></i>
              </button>
            </div>
            {showStickers && (
              <div className="absolute bottom-full left-0 right-0 bg-white border rounded-lg p-2 shadow-lg max-h-40 overflow-y-auto">
                <div className="grid grid-cols-3 gap-2">
                  {avatares.map((avatar) =>
                    avatar.stickers.map((sticker, index) => (
                      <button
                        key={`${avatar.id}-${index}`}
                        onClick={() => handleSendSticker(sticker)}
                        className="p-1 hover:bg-gray-100 rounded"
                      >
                        <img
                          src={sticker}
                          alt={`Sticker de ${avatar.nombre}`}
                          className="w-full h-12 object-contain"
                        />
                      </button>
                    ))
                  )}
                </div>
              </div>
            )}
          </div>
        </div>
      )}

      {installPrompt && (
        <div className="fixed bottom-24 left-4 right-4 bg-white p-4 rounded-lg shadow-lg flex items-center justify-between">
          <div className="text-sm">
            ¿Quieres instalar FitPoints en tu dispositivo?
          </div>
          <div className="flex space-x-2">
            <button
              onClick={() => setInstallPrompt(null)}
              className="px-3 py-1 text-sm text-gray-600"
            >
              No
            </button>
            <button
              onClick={() => {
                installPrompt.prompt();
                installPrompt.userChoice.then((choiceResult) => {
                  if (choiceResult.outcome === "accepted") {
                    setInstallPrompt(null);
                  }
                });
              }}
              className="px-3 py-1 text-sm bg-blue-500 text-white rounded"
            >
              Instalar
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

export default MainComponent;
